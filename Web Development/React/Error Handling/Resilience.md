# Application Resilience & Monitoring: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Application Resilience & Monitoring in React is the discipline of designing, implementing, and observing React applications such that they continue to function—possibly in a degraded capacity—when failures occur, and that those failures are visible to developers for diagnosis and repair.

**Technical Definition:** Application Resilience encompasses architectural patterns (Error Boundaries, feature flags, graceful degradation), network strategies (exponential backoff retries, offline caching via Service Workers, network state listeners), and observability practices (client-side logging, telemetry, error reporting with tools like Sentry and LogRocket) that together ensure a React application degrades gracefully under adverse conditions rather than failing catastrophically. The core principle is that a single runtime exception can unmount the entire React tree without boundaries, resulting in a blank screen with no logging, no context, and no recovery path. Resilience engineering treats error boundaries as architectural containment layers that define where failures stop propagating, monitoring makes failures visible, and recovery flows restore trust.

**Beginner-Friendly Explanation:** Sometimes things go wrong in a web app—the server goes down, the network drops, or a component crashes. Without planning, one small problem can break the entire app. Application Resilience is about building your app so that when something breaks, the rest keeps working. Monitoring is about making sure you (the developer) find out about the problem so you can fix it. Together, they make your app robust and reliable.

### Key Characteristics

- **Layered Containment:** Failures are contained at multiple levels—global boundaries, route-level boundaries, and feature-level boundaries—so a localized failure cannot escalate into a systemic outage.
- **Proactive Degradation:** Non-essential features (e.g., recommendation widgets, analytics trackers) can be disabled remotely via feature flags to keep the core application functional.
- **Adaptive Retries:** Transient network failures are retried automatically with exponential backoff, while permanent failures (4xx) are not retried.
- **Offline Awareness:** Applications detect network state changes and serve cached data when offline, synchronising when connectivity returns.
- **Observability by Design:** Errors are captured with stack traces, breadcrumbs, component stacks, and user session context, enabling rapid diagnosis.
- **Recovery-Oriented UX:** Users are given clear recovery paths—"Try Again" buttons, retry workflows, and graceful fallbacks—rather than being stranded on a broken screen.

### Prerequisites

- Solid understanding of React components, Hooks, and the component lifecycle.
- Knowledge of Error Boundaries and how they catch rendering errors.
- Familiarity with Promises, async/await, and HTTP status codes.
- Basic understanding of Service Workers and the Cache API (for offline support).
- Awareness of observability concepts (logging, tracing, metrics).

### Related Programming Areas

- **Error Handling:** Error Boundaries, try/catch, and global error handlers.
- **Network Programming:** Fetch API, Axios, AbortController, and HTTP protocols.
- **Observability:** Sentry, LogRocket, OpenTelemetry, and distributed tracing.
- **State Management:** Caching strategies, optimistic updates, and offline queues.
- **DevOps & Feature Management:** Feature flags, kill switches, and canary releases.
- **User Experience:** Fallback UI design, retry flows, and offline experiences.

### Core Concepts / Features

1. Graceful Degradation
2. Automated Retry Strategies
3. Offline & Network States
4. Client-Side Logging & Telemetry

---

## Core Concept 1: Graceful Degradation

### Definitions

**Core Definition:** Graceful Degradation is the practice of designing a React application so that when a non-essential feature or subsystem fails, the core functionality remains available and usable.

**Technical Definition:** Graceful degradation is implemented through a combination of architectural patterns: layered Error Boundaries that contain rendering failures to specific subtrees, feature flags that allow non-essential features to be disabled remotely, and fallback UI components that provide meaningful alternatives when a feature is unavailable. When a widget throws an error during render, React propagates the error upward through its parents until it reaches the nearest Error Boundary. That boundary renders a fallback UI for that region only, while the rest of the component tree remains intact and usable. Feature flags enable conditional rendering of features, allowing a broken code path to be disabled without rolling back the entire deployment.

**Beginner-Friendly Explanation:** Imagine your app is like a car. If the air conditioning breaks, you can still drive the car—you just roll down the windows. Graceful degradation means that if a non-essential part of your app (like a recommendation widget or a chat sidebar) breaks, the main parts (like the product page or checkout) keep working. You show a friendly message in the broken area instead of letting the whole page crash.

### Purposes

- To keep the core application functional when non-essential features fail.
- To isolate rendering failures to specific UI regions using layered Error Boundaries.
- To disable broken features remotely via feature flags without redeploying.
- To provide meaningful fallback UI that informs users without alarming them.
- To prevent a single widget failure from cascading into a full-page crash.
- To maintain user trust by ensuring critical workflows (checkout, login, search) remain available.

### Syntax Rules and Structure

**General Syntax for Layered Error Boundaries:**
```jsx
function App() {
  return (
    <GlobalErrorBoundary fallback={<FullPageError />}>
      <Header />
      <RouteErrorBoundary fallback={<RouteError />}>
        <DashboardPage />
      </RouteErrorBoundary>
      <Footer />
    </GlobalErrorBoundary>
  );
}

function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>
      <ErrorBoundary fallback={<WidgetError name="Revenue" />}>
        <RevenueChart />
      </ErrorBoundary>
      <ErrorBoundary fallback={<WidgetError name="Orders" />}>
        <RecentOrders />
      </ErrorBoundary>
      <ErrorBoundary fallback={<WidgetError name="Activity" />}>
        <TeamActivity />
      </ErrorBoundary>
    </div>
  );
}
```

**Component Breakdown:**
- `GlobalErrorBoundary`: The outermost boundary. Catches anything that escapes lower-level boundaries.
- `RouteErrorBoundary`: Per-route isolation. A crash in one route does not affect other routes.
- `ErrorBoundary`: Feature-level boundaries for individual widgets.
- `fallback`: A component or element rendered when an error is caught.

**General Syntax for Feature-Flag-Driven Degradation:**
```jsx
import { useFeatureFlag } from './featureFlags';

function ProductPage() {
  const showRecommendations = useFeatureFlag('recommendations-enabled', true);

  return (
    <div>
      <ProductDetails />
      {showRecommendations && (
        <ErrorBoundary fallback={null}>
          <RecommendationsWidget />
        </ErrorBoundary>
      )}
      <AddToCartButton />
    </div>
  );
}
```

**Component Breakdown:**
- `useFeatureFlag('recommendations-enabled', true)`: Checks a feature flag with a default value.
- `showRecommendations`: If `false`, the recommendations widget is not rendered at all.
- `ErrorBoundary fallback={null}`: If the widget fails, it renders nothing rather than a broken UI.
- The rest of the product page remains functional regardless.

**Syntax Rules:**
- Place Error Boundaries at data-fetch granularity: around each independent section or widget.
- Use a three-layer architecture: global (last resort), route-level (page isolation), and feature-level (widget isolation).
- Feature flags should have sensible defaults so the app degrades gracefully if the flag service is unavailable.
- Fallback UI should be contextual: card-level errors get inline messages, page-level errors get full-page fallbacks.
- Non-essential features (analytics, recommendations, chat) should be wrapped in boundaries that render `null` or a minimal placeholder on failure.
- Critical features (checkout, authentication) should have robust boundaries with recovery options.

**Constraints and Limitations:**
- Error Boundaries do not catch errors in event handlers, asynchronous code, or server-side rendering.
- Adding too many boundaries can fragment the user experience; use them strategically.
- Feature flags add complexity; ensure they are cleaned up after features are fully rolled out.
- A fallback that itself throws will propagate to the next boundary above it.
- Feature flag services may be unavailable; always have local defaults.

### Annotated Code Examples

**Example 1: Three-Layer Boundary Architecture with Feature Flags**

```jsx
import React from 'react';
import { ErrorBoundary } from 'react-error-boundary';

// Layer 1: Global fallback (last resort)
function GlobalError({ error, resetErrorBoundary }) {
  return (
    <div style={{ padding: '40px', textAlign: 'center' }}>
      <h1>Application Error</h1>
      <p>An unexpected error occurred. Please reload the page.</p>
      <button onClick={() => window.location.reload()}>Reload</button>
    </div>
  );
}

// Layer 2: Route fallback
function RouteError({ error, resetErrorBoundary }) {
  return (
    <div style={{ padding: '20px', border: '2px solid red' }}>
      <h2>Page Error</h2>
      <p>{error.message}</p>
      <button onClick={resetErrorBoundary}>Try Again</button>
    </div>
  );
}

// Layer 3: Feature fallback (minimal, inline)
function FeatureError({ error, resetErrorBoundary }) {
  return (
    <div role="alert" style={{ padding: '12px', border: '1px solid orange', borderRadius: '4px' }}>
      <p style={{ color: '#666', fontSize: '14px' }}>Widget unavailable: {error.message}</p>
      <button onClick={resetErrorBoundary} style={{ fontSize: '12px' }}>Retry Widget</button>
    </div>
  );
}

// Feature flag hook (simplified)
function useFeatureFlag(flagName, defaultValue = false) {
  // In production, this would fetch from a flag service
  const flags = { 'recommendations': false, 'chat': true };
  return flags[flagName] ?? defaultValue;
}

// Components that may fail
function BrokenWidget() {
  throw new Error('Data source unreachable');
}
function WorkingWidget() {
  return <p>Widget loaded successfully.</p>;
}

// Dashboard route with feature-level boundaries
function DashboardRoute() {
  const showRecommendations = useFeatureFlag('recommendations', true);
  const showChat = useFeatureFlag('chat', true);

  return (
    <div>
      <h2>Dashboard</h2>

      <ErrorBoundary FallbackComponent={FeatureError}>
        <BrokenWidget />
      </ErrorBoundary>

      <ErrorBoundary FallbackComponent={FeatureError}>
        <WorkingWidget />
      </ErrorBoundary>

      {showRecommendations && (
        <ErrorBoundary fallback={null}>
          <div>Recommendations would appear here</div>
        </ErrorBoundary>
      )}

      {showChat && (
        <ErrorBoundary FallbackComponent={FeatureError}>
          <div>Chat widget</div>
        </ErrorBoundary>
      )}
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary FallbackComponent={GlobalError}>
      <header>App Header</header>
      <ErrorBoundary FallbackComponent={RouteError}>
        <DashboardRoute />
      </ErrorBoundary>
      <footer>App Footer</footer>
    </ErrorBoundary>
  );
}

export default App;
```

**Expected Output:** The app displays the header, "Dashboard", a feature-level error for `BrokenWidget` ("Widget unavailable: Data source unreachable" with a "Retry Widget" button), "Widget loaded successfully." for `WorkingWidget`, and the chat widget. Recommendations are not rendered because the feature flag is `false`. The header and footer remain visible. The route-level and global boundaries are not triggered.

**Why This Output Occurs:** The three-layer architecture provides defence in depth. The feature-level Error Boundary around `BrokenWidget` catches its error first. The feature flag for recommendations is `false`, so that widget is not rendered at all. The chat widget is enabled and renders successfully. The header and footer are outside the route boundary and remain unaffected. This demonstrates both layered error containment and feature-flag-driven degradation.

### Real-World Cases

- **SaaS dashboards:** Placing Error Boundaries around each widget (revenue chart, recent orders, team activity) so one failed widget doesn't take down the entire dashboard.
- **E-commerce product pages:** Disabling the recommendation carousel via feature flag if it causes performance issues, while keeping product details and add-to-cart functional.
- **Social media platforms:** Wrapping each post or story in an Error Boundary to isolate malformed content.
- **Financial applications:** Disabling non-critical analytics widgets during market volatility to prioritise core trading functionality.
- **Content platforms:** Showing a simplified article view if the rich media player fails to load.

---

## Core Concept 2: Automated Retry Strategies

### Definitions

**Core Definition:** Automated Retry Strategies are systematic approaches to re-attempting failed network requests, typically using exponential backoff to avoid overwhelming the server and to improve the likelihood of success for transient failures.

**Technical Definition:** Exponential backoff is a retry algorithm where the delay between retry attempts increases exponentially (e.g., 1s, 2s, 4s, 8s), often with a maximum cap (e.g., 30 seconds) and jitter to prevent thundering herd problems. TanStack Query provides built-in retry support: queries that fail are silently retried 3 times, with exponential backoff delay before capturing and displaying an error to the UI. The default `retryDelay` doubles with each attempt, starting at 1000ms, and does not exceed 30 seconds. Custom retry logic can be provided via a function that receives the failure count and error, allowing retries to be skipped for 4xx errors (permanent failures) and applied only to 5xx errors and network failures.

**Beginner-Friendly Explanation:** Imagine you're trying to call a friend but the line is busy. You don't keep redialling instantly—you wait a few seconds, then a bit longer, then even longer. That's exponential backoff. In React, when a data request fails because the server is temporarily overloaded or the network hiccuped, the app automatically retries after increasing delays. This gives the server time to recover and increases the chance the request will succeed.

### Purposes

- To recover from transient network failures without user intervention.
- To avoid overwhelming a struggling server with immediate repeated requests.
- To distinguish between permanent failures (4xx) and transient failures (5xx, network errors).
- To provide a configurable retry policy at both global and per-query levels.
- To integrate with observability tools to log retry attempts and final failures.
- To improve perceived reliability without compromising user experience.

### Syntax Rules and Structure

**General Syntax with TanStack Query:**
```jsx
import { useQuery, QueryClient, QueryClientProvider } from '@tanstack/react-query';

// Global configuration
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error) => {
        // Do not retry on 4xx client errors
        if (error?.status >= 400 && error?.status < 500) return false;
        // Retry up to 3 times for other errors
        return failureCount < 3;
      },
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    },
  },
});

// Per-query override
function useUserData(userId) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    retry: 5,
    retryDelay: (attemptIndex) => Math.min(500 * 2 ** attemptIndex, 10000),
  });
}
```

**Component Breakdown:**
- `retry`: A boolean, number, or function that determines whether to retry.
- `retry: (failureCount, error) => ...`: Custom logic based on the failure count and error object.
- `retryDelay`: A function or integer that determines the delay before the next retry.
- `Math.min(1000 * 2 ** attemptIndex, 30000)`: Exponential backoff capped at 30 seconds.
- `attemptIndex`: Starts at 0 for the first retry.

**General Syntax for Custom Retry with Jitter:**
```jsx
function retryDelayWithJitter(attemptIndex) {
  const baseDelay = Math.min(1000 * 2 ** attemptIndex, 30000);
  // Add random jitter between 0 and 50% of the base delay
  const jitter = Math.random() * 0.5 * baseDelay;
  return baseDelay + jitter;
}

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retryDelay: retryDelayWithJitter,
    },
  },
});
```

**Component Breakdown:**
- `baseDelay`: Exponential backoff calculation.
- `jitter`: Random additional delay to prevent synchronized retries.
- `retryDelayWithJitter`: Returns the final delay with jitter applied.

**Syntax Rules:**
- Use `retry: false` to disable retries entirely for queries where retrying is inappropriate (e.g., 404s).
- Use `retry: (failureCount, error) => ...` to implement custom retry logic based on status codes.
- The default retry count is 3; the default delay starts at 1000ms and doubles with each attempt.
- On the server, retries default to 0 to make server rendering as fast as possible.
- Consider adding jitter to retry delays to prevent synchronized retries from multiple clients.
- Log retry attempts and final failures to your monitoring service for visibility.

**Constraints and Limitations:**
- Retrying 4xx errors (except 408 Request Timeout and 429 Too Many Requests) is generally futile and wastes resources.
- Infinite retries can cause memory leaks and performance issues; always set a maximum.
- Retries during server-side rendering should be disabled or minimised for performance.
- Retries respect the browser tab's focus state; retries pause when the tab is inactive unless configured otherwise.
- Custom retry logic must handle the shape of the error object, which varies by HTTP client.

### Annotated Code Examples

**Example 1: TanStack Query with Exponential Backoff and Error Classification**

```jsx
import React from 'react';
import { useQuery, QueryClient, QueryClientProvider } from '@tanstack/react-query';

async function fetchPosts() {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts');
  if (!response.ok) {
    const error = new Error(`HTTP ${response.status}`);
    error.status = response.status;
    throw error;
  }
  return response.json();
}

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error) => {
        // Do not retry on 4xx client errors
        if (error.status >= 400 && error.status < 500) {
          return false;
        }
        // Retry up to 5 times for 5xx and network errors
        return failureCount < 5;
      },
      retryDelay: (attemptIndex) => {
        const delay = Math.min(1000 * 2 ** attemptIndex, 30000);
        console.log(`Retry attempt ${attemptIndex + 1}: waiting ${delay}ms`);
        return delay;
      },
    },
  },
});

function PostList() {
  const { data, isError, error, isLoading, failureCount } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts,
  });

  if (isLoading) return <p>Loading posts...</p>;

  if (isError) {
    return (
      <div role="alert">
        <h2>Failed to load posts</h2>
        <p>{error.message}</p>
        <p>Retries attempted: {failureCount}</p>
      </div>
    );
  }

  return (
    <ul>
      {data.slice(0, 5).map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <PostList />
    </QueryClientProvider>
  );
}

export default App;
```

**Expected Output:** If the server returns a 500 error, the query retries up to 5 times with increasing delays (1s, 2s, 4s, 8s, 16s). The console logs each retry attempt with the delay. After all retries fail, "Failed to load posts" with the error message and "Retries attempted: 5" is displayed. If the server returns a 404, the query fails immediately without retries.

**Why This Output Occurs:** The `retry` function checks `error.status`. For 4xx errors, it returns `false`, disabling retries. For 5xx and network errors, it returns `true` for the first 5 attempts. The `retryDelay` function calculates exponential backoff capped at 30 seconds. The `failureCount` from `useQuery` reports how many retries were attempted. This pattern ensures that permanent failures are not retried, while transient failures get multiple chances to succeed.

**Example 2: Custom Retry with Circuit Breaker Pattern**

```jsx
import React from 'react';
import { useQuery, QueryClient, QueryClientProvider } from '@tanstack/react-query';

// Circuit breaker state
const circuitBreaker = {
  failures: 0,
  lastFailureTime: null,
  threshold: 5,
  resetTimeout: 60000, // 1 minute
};

async function fetchData() {
  // Check if circuit is open
  if (circuitBreaker.failures >= circuitBreaker.threshold) {
    const timeSinceLastFailure = Date.now() - circuitBreaker.lastFailureTime;
    if (timeSinceLastFailure < circuitBreaker.resetTimeout) {
      throw new Error('Circuit breaker open — service unavailable');
    }
    // Half-open: allow one request to test
    circuitBreaker.failures = 0;
  }

  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/posts');
    if (!response.ok) throw new Error(`HTTP ${response.status}`);

    // Reset on success
    circuitBreaker.failures = 0;
    return response.json();
  } catch (error) {
    circuitBreaker.failures++;
    circuitBreaker.lastFailureTime = Date.now();
    throw error;
  }
}

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error) => {
        // Do not retry if circuit is open
        if (error.message.includes('Circuit breaker open')) return false;
        return failureCount < 3;
      },
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 10000),
    },
  },
});

function DataView() {
  const { data, isError, error, isLoading } = useQuery({
    queryKey: ['data'],
    queryFn: fetchData,
    refetchInterval: 30000,
  });

  if (isLoading) return <p>Loading...</p>;
  if (isError) {
    return (
      <div role="alert">
        <p>Error: {error.message}</p>
        {error.message.includes('Circuit breaker') && (
          <p>Service temporarily unavailable. Please try again later.</p>
        )}
      </div>
    );
  }

  return <p>Data loaded: {data.length} items</p>;
}

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <DataView />
    </QueryClientProvider>
  );
}

export default App;
```

**Expected Output:** After 5 consecutive failures, the circuit breaker opens, and subsequent requests fail immediately with "Circuit breaker open — service unavailable." After 1 minute, the circuit enters a half-open state, allowing one request to test if the service has recovered. If that request succeeds, the circuit resets.

**Why This Output Occurs:** The circuit breaker tracks consecutive failures. When the failure count reaches the threshold (5), it stops allowing requests for the reset timeout period (60 seconds). This prevents the application from continuously hammering a failing service, giving it time to recover. The `retry` function checks for the circuit breaker message and disables retries when the circuit is open. This is a resilience pattern that complements exponential backoff.

### Real-World Cases

- **E-commerce checkout:** Retrying payment gateway requests with exponential backoff, while not retrying invalid card errors (4xx).
- **Dashboard data loading:** Retrying failed API calls for analytics data with a circuit breaker to avoid overwhelming a struggling backend.
- **Mobile applications:** Using aggressive retry policies for intermittent network conditions, with jitter to prevent synchronized retries.
- **Real-time feeds:** Using `refetchInterval` with retry logic for polling data sources.
- **Third-party API integration:** Retrying external API calls with backoff, while respecting rate limits (429 status codes).

---

## Core Concept 3: Offline & Network States

### Definitions

**Core Definition:** Offline & Network State management is the practice of detecting network connectivity changes in a React application, serving cached data when offline, and synchronising with the server when connectivity is restored.

**Technical Definition:** Offline support in React applications is implemented through two complementary mechanisms: (1) Service Workers that intercept network requests and serve cached responses, enabling the application shell and static assets to load without connectivity, and (2) network state listeners (`navigator.onLine`, `online`/`offline` events) that inform the application of connectivity changes so it can adjust its behaviour. The `useOffline` custom Hook tracks `navigator.onLine` and listens for `online` and `offline` events, returning an `isOffline` boolean. TanStack Query's `OnlineManager` provides a more sophisticated approach, detecting network status and automatically refetching queries when connectivity is restored. Offline data is persisted using `localStorage` or IndexedDB, and an offline-aware fetch utility can return cached data when the network is unavailable.

**Beginner-Friendly Explanation:** When you lose internet connection, most web apps break. Offline support means your app can still show you previously loaded content—like a cached list of items or your last viewed page—and queue up any changes you make until you're back online. It's like having a notebook where you write down what you want to do when you get back to the office.

### Purposes

- To detect when the user's device loses or regains network connectivity.
- To serve cached data when the network is unavailable, keeping the app usable.
- To queue user actions (form submissions, mutations) for later synchronisation.
- To automatically refetch data when connectivity is restored.
- To provide clear UI feedback about the current network state.
- To enable progressive web app (PWA) experiences with offline capabilities.

### Syntax Rules and Structure

**General Syntax for Network State Hook:**
```jsx
import { useState, useEffect } from 'react';

export function useOffline() {
  const [isOffline, setIsOffline] = useState(!navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOffline(false);
    const handleOffline = () => setIsOffline(true);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOffline;
}
```

**Component Breakdown:**
- `useState(!navigator.onLine)`: Initialises with the current network status.
- `handleOnline` / `handleOffline`: Event handlers that update the state.
- `window.addEventListener('online', ...)`: Subscribes to connectivity changes.
- Cleanup function removes the event listeners.

**General Syntax for Offline Storage Service:**
```javascript
const STORAGE_KEY = 'offline_data';

export const offlineStorage = {
  save: (key, data) => {
    const store = JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}');
    store[key] = { data, timestamp: Date.now() };
    localStorage.setItem(STORAGE_KEY, JSON.stringify(store));
  },
  get: (key) => {
    const store = JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}');
    return store[key]?.data || null;
  },
  remove: (key) => {
    const store = JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}');
    delete store[key];
    localStorage.setItem(STORAGE_KEY, JSON.stringify(store));
  },
  clear: () => {
    localStorage.removeItem(STORAGE_KEY);
  },
};
```

**Component Breakdown:**
- `save(key, data)`: Stores data with a timestamp.
- `get(key)`: Retrieves stored data.
- `remove(key)`: Deletes specific data.
- `clear()`: Clears all stored data.

**General Syntax for Offline-Aware Fetch:**
```javascript
import { offlineStorage } from './offlineStorage';

export async function offlineFetch(url, options = {}) {
  try {
    const response = await fetch(url, options);
    const data = await response.json();
    // Cache successful response
    offlineStorage.save(url, data);
    return data;
  } catch (error) {
    // If offline, return cached data
    if (!navigator.onLine) {
      const cachedData = offlineStorage.get(url);
      if (cachedData) {
        return cachedData;
      }
    }
    throw error;
  }
}
```

**Component Breakdown:**
- `try` block: Fetches data and caches it on success.
- `catch` block: If offline and cached data exists, returns it.
- `offlineStorage.save(url, data)`: Caches the response.
- `offlineStorage.get(url)`: Retrieves cached data.

**Syntax Rules:**
- Use `navigator.onLine` for initial state, but listen for `online`/`offline` events for updates.
- Cache API responses using `localStorage` (small data) or IndexedDB (large data).
- Always provide a fallback when cached data is unavailable.
- Clean up event listeners in the `useEffect` cleanup function.
- Consider using TanStack Query's `OnlineManager` for automatic refetching on reconnection.
- Service Workers are required for caching static assets and enabling offline navigation.

**Constraints and Limitations:**
- `navigator.onLine` can return false positives for apps loaded via Service Workers that can work without internet.
- `localStorage` has a size limit (typically 5–10 MB) and is synchronous, which can block the main thread.
- IndexedDB is asynchronous and larger, but more complex to use.
- Service Workers require HTTPS (except on localhost) and have a lifecycle that can be confusing.
- Offline caching strategies must consider cache invalidation and staleness.
- TanStack Query's `OnlineManager` may not work correctly with Service Worker-loaded apps.

### Annotated Code Examples

**Example 1: Offline-Aware Data Fetching with Cache Fallback**

```jsx
import React, { useState, useEffect } from 'react';
import { useOffline } from './hooks/useOffline';
import { offlineFetch } from './utils/offlineFetch';

function UserList() {
  const [users, setUsers] = useState([]);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);
  const isOffline = useOffline();

  useEffect(() => {
    let ignore = false;

    async function fetchUsers() {
      try {
        setLoading(true);
        setError(null);

        const data = await offlineFetch(
          'https://jsonplaceholder.typicode.com/users'
        );

        if (!ignore) setUsers(data);
      } catch (err) {
        if (!ignore) setError(err.message);
      } finally {
        if (!ignore) setLoading(false);
      }
    }

    fetchUsers();

    return () => { ignore = true; };
  }, [isOffline]); // Re-fetch when network status changes

  if (loading) return <p>Loading users...</p>;

  if (error) {
    return (
      <div role="alert">
        <p>Error: {error}</p>
        {isOffline && <p>You are offline. Showing cached data if available.</p>}
      </div>
    );
  }

  return (
    <div>
      {isOffline && (
        <div style={{ padding: '8px', backgroundColor: '#fff3cd', marginBottom: '10px' }}>
          You are offline. Data may be stale.
        </div>
      )}
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name} — {user.email}</li>
        ))}
      </ul>
    </div>
  );
}

export default UserList;
```

**Expected Output:** When online, the user list loads from the API and is cached. When offline, the cached user list is displayed with a yellow banner "You are offline. Data may be stale." If no cached data exists, an error message is shown. When connectivity is restored, the list automatically refetches.

**Why This Output Occurs:** The `useOffline` Hook tracks network status. The `offlineFetch` utility attempts to fetch from the API and caches the result. If the network is unavailable, it falls back to cached data. The `useEffect` dependency on `isOffline` causes a refetch when connectivity changes, ensuring the data is refreshed when the user comes back online.

**Example 2: TanStack Query with OnlineManager for Automatic Refetch**

```jsx
import React from 'react';
import {
  useQuery,
  QueryClient,
  QueryClientProvider,
  onlineManager,
} from '@tanstack/react-query';
import { useEffect } from 'react';

// Custom OnlineManager integration
function useOnlineManager() {
  useEffect(() => {
    // Set initial state
    onlineManager.setOnline(navigator.onLine);

    const handleOnline = () => onlineManager.setOnline(true);
    const handleOffline = () => onlineManager.setOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
}

async function fetchNotifications() {
  const response = await fetch('/api/notifications');
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  return response.json();
}

function NotificationList() {
  useOnlineManager();

  const { data, isError, error, isLoading, isFetching, refetch } = useQuery({
    queryKey: ['notifications'],
    queryFn: fetchNotifications,
    retry: 3,
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
  });

  if (isLoading) return <p>Loading notifications...</p>;

  if (isError) {
    return (
      <div role="alert">
        <p>Error: {error.message}</p>
        <button onClick={() => refetch()}>Retry</button>
      </div>
    );
  }

  return (
    <div>
      {isFetching && <p>Refreshing...</p>}
      <ul>
        {data?.map(notification => (
          <li key={notification.id}>{notification.message}</li>
        ))}
      </ul>
    </div>
  );
}

function App() {
  const queryClient = new QueryClient();

  return (
    <QueryClientProvider client={queryClient}>
      <NotificationList />
    </QueryClientProvider>
  );
}

export default App;
```

**Expected Output:** The notification list loads when online. When the user goes offline, TanStack Query pauses refetches. When connectivity is restored, TanStack Query automatically refetches the notifications, and "Refreshing..." appears briefly.

**Why This Output Occurs:** The `useOnlineManager` Hook integrates the browser's network events with TanStack Query's `onlineManager`. When `onlineManager.setOnline(true)` is called, TanStack Query automatically refetches queries that were paused due to being offline. This provides a seamless experience where data is refreshed as soon as connectivity returns.

### Real-World Cases

- **Progressive Web Apps (PWAs):** Caching the app shell and data for offline use, enabling installation on mobile devices.
- **Field service applications:** Allowing technicians to view and update work orders without connectivity, syncing when back online.
- **E-commerce carts:** Preserving cart contents when offline and syncing when connectivity returns.
- **Note-taking apps:** Allowing users to create and edit notes offline, syncing changes when online.
- **Travel applications:** Caching itineraries and maps for offline access.
- **Social media apps:** Showing cached feed content when offline with an indicator that data may be stale.

---

## Core Concept 4: Client-Side Logging & Telemetry

### Definitions

**Core Definition:** Client-Side Logging & Telemetry is the practice of capturing, enriching, and transmitting error events, performance metrics, and user interaction data from the browser to a monitoring service for analysis and alerting.

**Technical Definition:** Client-side telemetry involves integrating error reporting tools (Sentry, LogRocket, OpenTelemetry) into the React application. When an error is thrown, the SDK builds an event containing the stack trace, a list of breadcrumbs (console logs, clicks, XHR/fetch calls), the current URL, and any context you attach. Breadcrumbs create a trail of events that happened prior to an issue, providing critical context for debugging. Sentry's React SDK provides an `ErrorBoundary` component that automatically captures React component errors, error boundary crashes, and component stack traces. Integration involves initialising the SDK with a DSN, configuring integrations (tracing, session replay, logs), and wrapping the application with the provided Error Boundary.

**Beginner-Friendly Explanation:** When something breaks in your app, you want to know about it—not just that it broke, but what the user was doing, what they clicked, and what the error message was. Client-side telemetry tools like Sentry automatically capture all this information and send it to a dashboard where you can see exactly what happened. It's like having a flight recorder (black box) for your web app.

### Purposes

- To capture error events with full stack traces and component stacks.
- To record breadcrumbs—a trail of user actions leading up to an error.
- To attach user session context (user ID, route, device) for debugging.
- To monitor application performance and track regressions.
- To receive alerts when error rates spike.
- To enable session replay for visual reproduction of user issues.

### Syntax Rules and Structure

**General Syntax for Sentry Initialization:**
```javascript
// instrument.js — must be imported before any other imports
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: 'YOUR_SENTRY_DSN',
  integrations: [
    Sentry.browserTracingIntegration(),
    Sentry.replayIntegration(),
  ],
  // Performance monitoring
  tracesSampleRate: 0.1,
  // Session replay
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
  // Environment
  environment: process.env.NODE_ENV,
  // Attach user context
  beforeSend(event) {
    // Filter or modify events before sending
    return event;
  },
});
```

**Component Breakdown:**
- `dsn`: The Data Source Name that identifies your Sentry project.
- `integrations`: Array of enabled integrations (tracing, replay, etc.).
- `tracesSampleRate`: Percentage of transactions to capture for performance monitoring.
- `replaysSessionSampleRate`: Percentage of sessions to record for replay.
- `replaysOnErrorSampleRate`: Percentage of error sessions to record.
- `environment`: The deployment environment (development, staging, production).

**General Syntax for Sentry Error Boundary:**
```jsx
import * as Sentry from '@sentry/react';

function App() {
  return (
    <Sentry.ErrorBoundary
      fallback={({ error, resetError }) => (
        <div role="alert">
          <h2>Something went wrong</h2>
          <p>{error.message}</p>
          <button onClick={resetError}>Try again</button>
        </div>
      )}
      onError={(error, componentStack) => {
        console.error('Error caught by Sentry boundary:', error, componentStack);
      }}
      showDialog
    >
      <MyComponent />
    </Sentry.ErrorBoundary>
  );
}
```

**Component Breakdown:**
- `Sentry.ErrorBoundary`: A pre-built Error Boundary from Sentry.
- `fallback`: A render function receiving `error` and `resetError`.
- `onError`: Callback for additional error handling.
- `showDialog`: Shows a Sentry user feedback dialog when an error occurs.

**General Syntax for Manual Breadcrumbs:**
```javascript
import * as Sentry from '@sentry/react';

// Add a custom breadcrumb
Sentry.addBreadcrumb({
  category: 'user-action',
  message: 'User clicked checkout button',
  level: 'info',
  data: {
    cartValue: 99.99,
    itemCount: 3,
  },
});

// Capture a custom error
Sentry.captureException(new Error('Payment processing failed'), {
  tags: { payment_provider: 'stripe' },
  extra: { orderId: '12345' },
});
```

**Component Breakdown:**
- `Sentry.addBreadcrumb()`: Adds a breadcrumb to the trail.
- `category`: A label for grouping breadcrumbs.
- `message`: A human-readable description.
- `level`: Severity level (info, warning, error, etc.).
- `data`: Additional structured data.
- `Sentry.captureException()`: Captures a custom error with tags and extra data.

**Syntax Rules:**
- Initialise Sentry before the React tree mounts, in a file imported at the entry point.
- Use `Sentry.ErrorBoundary` alongside or instead of custom Error Boundaries.
- Add manual breadcrumbs for important user actions (checkout, login, form submission).
- Set `tracesSampleRate` appropriately: 0.1 (10%) for production, 1.0 for development.
- Use `beforeSend` to filter out sensitive data or development-only errors.
- Configure `replaysOnErrorSampleRate: 1.0` to always record sessions where errors occur.

**Constraints and Limitations:**
- Sentry adds bundle size; use tree-shaking and dynamic imports for optional features.
- Sending too many events can exhaust your Sentry quota; use sampling and filtering.
- Session replay may capture sensitive user data; configure privacy options (masking, blocking).
- Breadcrumbs have a maximum limit (typically 100); older breadcrumbs are dropped.
- OpenTelemetry provides more granular tracing but requires more setup and backend infrastructure.
- LogRocket provides session replay but is a paid service with usage limits.

### Annotated Code Examples

**Example 1: Sentry Integration with Error Boundary and Breadcrumbs**

```jsx
// instrument.js (import this first in main.jsx or index.js)
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: 'YOUR_SENTRY_DSN',
  integrations: [
    Sentry.browserTracingIntegration(),
    Sentry.replayIntegration({
      maskAllText: false,
      blockAllMedia: false,
    }),
  ],
  tracesSampleRate: 0.2,
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
  environment: 'production',
});

// App.jsx
import React from 'react';
import * as Sentry from '@sentry/react';

function CheckoutButton({ cart }) {
  function handleClick() {
    // Add breadcrumb for the user action
    Sentry.addBreadcrumb({
      category: 'user-action',
      message: 'Checkout button clicked',
      level: 'info',
      data: {
        itemCount: cart.length,
        total: cart.reduce((sum, item) => sum + item.price, 0),
      },
    });

    try {
      processCheckout(cart);
    } catch (error) {
      // Capture the error with context
      Sentry.captureException(error, {
        tags: { action: 'checkout' },
        extra: { cartItems: cart },
      });
      // Re-throw to let Error Boundary handle UI
      throw error;
    }
  }

  return <button onClick={handleClick}>Checkout</button>;
}

function processCheckout(cart) {
  if (cart.length === 0) {
    throw new Error('Cannot checkout with empty cart');
  }
  // Simulate processing
  console.log('Processing checkout...');
}

function ErrorFallback({ error, resetError }) {
  return (
    <div role="alert" style={{ padding: '20px', textAlign: 'center' }}>
      <h2>Checkout Error</h2>
      <p>{error.message}</p>
      <button onClick={resetError}>Try Again</button>
    </div>
  );
}

function App() {
  const cart = [
    { id: 1, name: 'Product A', price: 29.99 },
    { id: 2, name: 'Product B', price: 49.99 },
  ];

  return (
    <Sentry.ErrorBoundary
      fallback={ErrorFallback}
      onError={(error, componentStack) => {
        console.error('Error caught:', error);
        console.error('Component stack:', componentStack);
      }}
      showDialog
    >
      <h1>Checkout</h1>
      <CheckoutButton cart={cart} />
    </Sentry.ErrorBoundary>
  );
}

export default App;
```

**Expected Output:** When the checkout button is clicked, a breadcrumb is added. If the cart is empty, an error is thrown, captured by Sentry with tags and extra data, and the Error Boundary displays "Checkout Error" with the error message and a "Try Again" button. A Sentry feedback dialog may also appear.

**Why This Output Occurs:** The `Sentry.addBreadcrumb()` call records the user's action, providing context if an error occurs. The `Sentry.captureException()` call captures the error with custom tags and extra data. The `Sentry.ErrorBoundary` catches the thrown error and renders the fallback. The `onError` callback logs additional information. This provides comprehensive telemetry for debugging checkout failures.

**Example 2: Manual Breadcrumbs and Session Context**

```jsx
import React from 'react';
import * as Sentry from '@sentry/react';

function LoginForm() {
  const [email, setEmail] = React.useState('');
  const [password, setPassword] = React.useState('');

  function handleSubmit(e) {
    e.preventDefault();

    // Add breadcrumb before attempting login
    Sentry.addBreadcrumb({
      category: 'auth',
      message: 'Login attempt',
      level: 'info',
      data: { email: email.replace(/(.{2}).*@/, '$1***@') }, // Partial masking
    });

    try {
      // Simulate login
      if (password.length < 8) {
        throw new Error('Password too short');
      }

      // Set user context on successful login
      Sentry.setUser({
        id: 'user-123',
        email: email,
        username: email.split('@')[0],
      });

      Sentry.addBreadcrumb({
        category: 'auth',
        message: 'Login successful',
        level: 'info',
      });
    } catch (error) {
      // Capture login error
      Sentry.captureException(error, {
        tags: { auth_action: 'login' },
        level: 'warning',
      });
      alert(error.message);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
}

export default LoginForm;
```

**Expected Output:** When the login form is submitted, a breadcrumb is added. If the password is too short, an error is captured with the tag `auth_action: login` and a warning level. On successful login, the user context is set in Sentry, and a success breadcrumb is added. All these events appear in the Sentry dashboard with full context.

**Why This Output Occurs:** The `Sentry.addBreadcrumb()` calls create a trail of authentication events. The `Sentry.setUser()` call attaches user context to all subsequent events. The `Sentry.captureException()` call captures the login error with custom tags. This provides a complete picture of the authentication flow and any failures that occur.

### Real-World Cases

- **E-commerce platforms:** Tracking checkout errors, payment failures, and cart abandonment with breadcrumbs showing user actions.
- **SaaS applications:** Monitoring errors in dashboard widgets and capturing user context for debugging.
- **Financial applications:** Capturing authentication errors and transaction failures with strict privacy controls.
- **Healthcare applications:** Tracking errors while ensuring HIPAA compliance through data masking.
- **Gaming platforms:** Monitoring client-side crashes and performance issues with session replay.
- **Multi-tenant applications:** Using tags to separate errors by tenant or organisation.

---

## References

- Sentry for React – Official Documentation: https://docs.sentry.io/platforms/javascript/guides/react/
- Sentry – Breadcrumbs: https://docs.sentry.io/platforms/javascript/guides/react/enriching-events/breadcrumbs/
- Sentry – Error Boundaries: https://docs.sentry.io/platforms/javascript/guides/react/features/error-boundary/
- Sentry – Session Replay: https://docs.sentry.io/product/session-replay/
- TanStack Query – Query Retries: https://tanstack.com/query/latest/docs/framework/react/guides/query-retries
- TanStack Query – Important Defaults: https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults
- TanStack Query – OnlineManager: https://tanstack.com/query/latest/docs/framework/react/guides/network-mode
- CoreUI – How to add offline support in React: https://coreui.io/answers/how-to-add-offline-support-in-react/
- MDN Web Docs – `navigator.onLine`: https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine
- MDN Web Docs – Service Worker API: https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
- MDN Web Docs – Online and offline events: https://developer.mozilla.org/en-US/docs/Web/API/Window/online_event
- React Official Documentation – Error Boundaries: https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- react-error-boundary Library: https://github.com/bvaughn/react-error-boundary
- LogRocket – React Error Monitoring: https://logrocket.com/
- OpenTelemetry – JavaScript SDK: https://opentelemetry.io/docs/languages/js/
- Web.dev – Service Workers: https://web.dev/learn/pwa/service-workers
- Web.dev – Offline Cookbook: https://web.dev/articles/offline-cookbook
- Workbox – Precaching and Routing: https://developer.chrome.com/docs/workbox
- Educative – Error Monitoring and Recovery Hooks: https://www.educative.io/courses/learn-react/lta/error-monitoring-and-recovery-hooks
- GitNation – Designing for Failure: The Senior React Dev's Production Toolkit: https://gitnation.com/