# External APIs & Resiliency — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** External API integration is the practice of consuming third-party HTTP services (REST or GraphQL) from a Node.js application, while resiliency is the set of patterns — timeouts, retries, rate limiting, circuit breakers — that ensure the application remains functional when those external services fail, slow down, or become unavailable.

**Technical Definition:** This domain covers HTTP client selection and configuration (Axios, native Fetch, GraphQL clients), authentication mechanisms for outbound requests (Bearer tokens, API keys, AWS IAM Signature V4, mutual TLS), resilience patterns (timeouts, exponential backoff, rate limiting, circuit breakers), and webhook ingestion (signature verification, idempotency). The goal is to build integrations that degrade gracefully rather than cascade failures across the system.

**Beginner-Friendly Explanation:** When your app talks to another service (like Stripe for payments or GitHub for repository data), things can go wrong: the other service might be slow, down, or return errors. Resiliency patterns are like safety nets — they ensure your app doesn't crash when the external service has problems. You set timeouts so you don't wait forever, retry with increasing delays, and use a circuit breaker that stops trying when the service is clearly broken.

### Key Characteristics

- **Client choice matters:** Axios auto-throws on HTTP errors, Fetch does not; Got offers built-in retries and pagination.
- **Authentication diversity:** Bearer tokens, API keys, AWS SigV4, and mTLS each suit different trust models.
- **Resiliency is layered:** Timeouts + retries + circuit breakers + rate limiting work together.
- **Webhooks are push-based:** Third-party services send events to your server; you must verify authenticity and handle duplicates.
- **Idempotency is essential:** Network retries can cause duplicate operations; idempotency keys prevent double-charging or double-processing.

### Prerequisites

- **Node.js runtime** (v18 or higher for native Fetch).
- **Express.js installed:** `npm install express`.
- **Basic understanding of HTTP:** Methods, status codes, headers.
- **Familiarity with async/await and Promises.**
- **A registered API key or OAuth application** for the external service you're integrating.

### Related Programming Areas

- **Microservices:** Resiliency patterns are critical for service-to-service communication.
- **Payment processing:** Webhooks and idempotency are essential for Stripe, PayPal, etc.
- **DevOps:** AWS SigV4 for infrastructure API calls.
- **Security:** mTLS for zero-trust service meshes.
- **Observability:** Circuit breaker events and retry metrics feed into monitoring.

### Core Concepts

1. **REST & GraphQL Clients** — Axios, native Fetch, HTTP/2 streams.
2. **API Authentication** — Bearer tokens, API Keys, AWS IAM signing, mTLS.
3. **Resiliency Patterns** — Timeouts, exponential backoff, rate limiting, circuit breakers.
4. **Webhooks** — Secure receiving, signature verification, idempotency.

---

## Core Concept 1: REST & GraphQL Clients

### Definitions

**Core Definition:** An HTTP client is the library or built-in API used to send requests to external services; REST clients send standard HTTP requests to resource endpoints, while GraphQL clients send queries and mutations to a single endpoint.

**Technical Definition:** Node.js offers multiple HTTP client options: **Axios** (Promise-based, auto-parses JSON, throws on non-2xx status, supports interceptors), **native Fetch** (built into Node.js 18+, zero-dependency, requires manual `response.ok` checks), **Got** (Node.js-only, built-in retries, pagination, and streaming), and **undici** (maximum throughput, HTTP/2 support). GraphQL clients typically use `fetch` or `axios` to POST queries to a `/graphql` endpoint, or use dedicated libraries like `graphql-request` or Apollo Client.

**Beginner-Friendly Explanation:** Axios and Fetch are both ways to make HTTP requests. Axios is like a Swiss Army knife — it does more for you (auto-parses JSON, throws errors on 404/500). Fetch is like a basic tool — it's built into Node.js, but you have to check for errors yourself. For GraphQL, you send a query string and variables to one endpoint instead of hitting different URLs for different data.

### Purposes

- To send HTTP requests to external REST APIs or GraphQL endpoints.
- To choose the right client for the use case (interceptors, retries, streaming, zero-dependency).
- To handle HTTP/2 streams for efficient multiplexed communication.
- To parse and validate responses from external services.

### Sub-Feature 1.1: Axios vs. Native Fetch

#### Syntax Rules and Structure

| Feature | Axios | Native Fetch |
|---------|-------|--------------|
| Error on 4xx/5xx | Yes (throws automatically) | No (must check `response.ok`) |
| JSON parsing | Automatic | Requires `.json()` |
| Interceptors | Yes | No |
| Bundle size | ~40KB | 0KB (built-in) |
| HTTP/2 | Via `http2-wrapper` | Via Undici dispatcher |
| Streaming | Basic | Good (Web ReadableStream) |

**Axios Error Handling:**
```js
try {
  const { data } = await axios.get('/api/users/999');
} catch (error) {
  if (error.response) {
    console.log(error.response.status); // 404
    console.log(error.response.data);   // Error body
  }
}
```

**Fetch Error Handling:**
```js
const response = await fetch('/api/users/999');
if (!response.ok) {
  throw new Error(`HTTP error: ${response.status}`);
}
const data = await response.json();
```

#### Annotated Code Example

```js
// rest-clients.js
const axios = require('axios');

async function fetchWithAxios(url) {
  try {
    const { data, status } = await axios.get(url, { timeout: 5000 });
    return { success: true, status, data };
  } catch (error) {
    if (error.response) {
      return { success: false, status: error.response.status, error: 'HTTP error' };
    }
    return { success: false, error: error.message };
  }
}

async function fetchWithFetch(url) {
  try {
    const response = await fetch(url, { signal: AbortSignal.timeout(5000) });
    if (!response.ok) {
      return { success: false, status: response.status, error: 'HTTP error' };
    }
    const data = await response.json();
    return { success: true, status: response.status, data };
  } catch (error) {
    return { success: false, error: error.message };
  }
}

// Usage
fetchWithAxios('https://api.github.com/users/octocat')
  .then(result => console.log('Axios:', result.success));

fetchWithFetch('https://api.github.com/users/octocat')
  .then(result => console.log('Fetch:', result.success));
```

**Expected Output:**
```
Axios: true
Fetch: true
```

**Why this output:** Axios automatically parses the JSON response and throws an error for non-2xx statuses. Fetch requires an explicit `response.ok` check. Both return the same data for a successful request. The key difference is error handling: Axios's `catch` block only runs for network errors or non-2xx responses, while Fetch's `catch` only runs for network errors — HTTP errors must be checked manually.

### Sub-Feature 1.2: GraphQL Clients

#### Syntax Rules and Structure

```js
// Using fetch for GraphQL
const response = await fetch('https://api.example.com/graphql', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify({
    query: `
      query GetUser($id: ID!) {
        user(id: $id) {
          name
          email
        }
      }
    `,
    variables: { id: '123' }
  })
});
const { data, errors } = await response.json();
```

#### Annotated Code Example

```js
// graphql-client.js
const axios = require('axios');

async function graphqlQuery(endpoint, query, variables = {}) {
  const response = await axios.post(endpoint, {
    query,
    variables
  }, {
    headers: { 'Content-Type': 'application/json' },
    timeout: 10000
  });

  if (response.data.errors) {
    throw new Error(response.data.errors.map(e => e.message).join(', '));
  }

  return response.data.data;
}

// Usage
const query = `
  query GetUser($id: ID!) {
    user(id: $id) {
      name
      email
    }
  }
`;

graphqlQuery('https://api.example.com/graphql', query, { id: '123' })
  .then(data => console.log('User:', data.user.name))
  .catch(err => console.error('GraphQL error:', err.message));
```

**Expected Output:**
```
User: Alice Johnson
```

**Why this output:** GraphQL queries are sent as POST requests with a `query` string and `variables` object. The response contains either `data` (on success) or `errors` (on failure). The client must check for `errors` before accessing `data`.

---

## Core Concept 2: API Authentication

### Definitions

**Core Definition:** API authentication is the mechanism by which a client proves its identity to an external service, using credentials such as Bearer tokens, API keys, cryptographic signatures, or client certificates.

**Technical Definition:** External APIs use different authentication schemes: **Bearer tokens** (OAuth 2.0 access tokens sent in the `Authorization: Bearer <token>` header), **API keys** (static secrets sent in headers or query parameters), **AWS IAM Signature V4** (HMAC-based request signing using access key, secret key, and session token), and **mutual TLS (mTLS)** (both client and server present X.509 certificates during the TLS handshake).

**Beginner-Friendly Explanation:** Authentication is like showing ID at the door. Bearer tokens are like a temporary visitor pass. API keys are like a membership card. AWS SigV4 is like a notarised signature that proves the request came from you. mTLS is like both you and the server showing ID to each other before talking.

### Purposes

- To prove the client's identity to the external service.
- To authorise access to specific resources or operations.
- To prevent unauthorised use of the API.
- To enable service-to-service trust (mTLS, SigV4).

### Sub-Feature 2.1: Bearer Tokens

#### Syntax Rules and Structure

```js
const response = await fetch('https://api.example.com/data', {
  headers: {
    'Authorization': `Bearer ${accessToken}`
  }
});
```

| Component | Breakdown |
|-----------|-----------|
| `Authorization` | Standard header for credentials. |
| `Bearer` | Token type prefix. |
| `accessToken` | The OAuth 2.0 access token. |

**Rules:**
- Bearer tokens MUST be sent in the `Authorization` header, never in query strings.
- Tokens should be short-lived; use refresh tokens to obtain new ones.
- Always use HTTPS to prevent token interception.

---

### Sub-Feature 2.2: API Keys

#### Syntax Rules and Structure

```js
const response = await axios.get('https://api.example.com/data', {
  headers: {
    'X-API-Key': process.env.API_KEY
  }
});
```

| Header | Common Usage |
|--------|-------------|
| `X-API-Key` | Custom header for API keys. |
| `Authorization: ApiKey <key>` | Alternative format. |

**Rules:**
- Never commit API keys to source control; use environment variables.
- Rotate keys periodically.
- Use separate keys for different environments (dev, staging, prod).

---

### Sub-Feature 2.3: AWS IAM Signature V4

#### Definitions

**Core Definition:** AWS Signature Version 4 is a protocol for signing HTTP requests with AWS credentials to authenticate to AWS services.

**Technical Definition:** SigV4 uses a series of HMAC-SHA256 operations to create a signature from the request's canonical form (method, URL, headers, payload hash). The signature is included in the `Authorization` header along with the access key ID, signed headers, and timestamp.

#### Syntax Rules and Structure

```js
const { HttpRequest } = require('@smithy/protocol-http');
const { fromNodeProviderChain } = require('@aws-sdk/credential-providers');
const { SignatureV4 } = require('@smithy/signature-v4');
const { Sha256 } = require('@aws-crypto/sha256-universal');
const https = require('https');

const request = new HttpRequest({
  hostname: 'api.example.com',
  path: '/resource',
  method: 'POST',
  headers: {
    'host': 'api.example.com',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ data: 'value' })
});

const credentials = await fromNodeProviderChain()();
const signer = new SignatureV4({
  credentials,
  region: 'us-east-1',
  service: 'execute-api',
  sha256: Sha256
});

const signedRequest = await signer.sign(request);
```

**Rules:**
- The `service` must match the AWS service being called (e.g., `execute-api` for API Gateway).
- The `region` must match the target service's region.
- The payload must be hashed with SHA-256 and included in the signature.

#### Annotated Code Example

```js
// aws-sigv4.js
const { HttpRequest } = require('@smithy/protocol-http');
const { fromNodeProviderChain } = require('@aws-sdk/credential-providers');
const { SignatureV4 } = require('@smithy/signature-v4');
const { Sha256 } = require('@aws-crypto/sha256-universal');
const https = require('https');

async function signedRequest() {
  const request = new HttpRequest({
    hostname: 'my-api.execute-api.us-east-1.amazonaws.com',
    path: '/prod/data',
    method: 'GET',
    headers: { 'host': 'my-api.execute-api.us-east-1.amazonaws.com' }
  });

  const credentials = await fromNodeProviderChain()();
  const signer = new SignatureV4({
    credentials,
    region: 'us-east-1',
    service: 'execute-api',
    sha256: Sha256
  });

  const signed = await signer.sign(request);

  return new Promise((resolve, reject) => {
    const req = https.request({
      hostname: signed.hostname,
      port: 443,
      path: signed.path,
      method: signed.method,
      headers: signed.headers
    }, (res) => {
      let body = '';
      res.on('data', chunk => body += chunk);
      res.on('end', () => resolve({ status: res.statusCode, body }));
    });
    req.on('error', reject);
    req.end();
  });
}

signedRequest().then(result => console.log('Status:', result.status));
```

**Expected Output:**
```
Status: 200
```

**Why this output:** The `SignatureV4` signer creates a canonical request from the HTTP request, hashes it with SHA-256, and computes an HMAC-SHA256 signature using the AWS secret key. The signed headers are then sent to the AWS service, which verifies the signature using the access key's corresponding secret.

---

### Sub-Feature 2.4: Mutual TLS (mTLS)

#### Definitions

**Core Definition:** Mutual TLS is a TLS handshake where both the client and the server present X.509 certificates, providing bidirectional authentication.

**Technical Definition:** In standard TLS, only the server presents a certificate. In mTLS, the client also presents a certificate, which the server validates against a trusted CA. This is implemented in Node.js by passing `key`, `cert`, and `ca` options to an `https.Agent`.

#### Syntax Rules and Structure

```js
const https = require('https');
const fs = require('fs');
const axios = require('axios');

const httpsAgent = new https.Agent({
  key: fs.readFileSync('./client-key.pem'),
  cert: fs.readFileSync('./client-cert.pem'),
  ca: fs.readFileSync('./ca-cert.pem'),
  rejectUnauthorized: true
});

const response = await axios.get('https://mtls-api.example.com/data', {
  httpsAgent
});
```

| Option | Description |
|--------|-------------|
| `key` | Client's private key. |
| `cert` | Client's certificate. |
| `ca` | CA certificate to verify the server. |
| `rejectUnauthorized` | Reject invalid server certificates. |

**Rules:**
- Both client and server must have their own private key and certificate.
- The CA certificate is used to verify the other party's certificate.
- Axios has no special mTLS logic; all behaviour depends on Node's `https.Agent` and the TLS protocol.

#### Annotated Code Example

```js
// mtls-client.js
const https = require('https');
const fs = require('fs');
const axios = require('axios');

const httpsAgent = new https.Agent({
  key: fs.readFileSync('./certs/client-key.pem'),
  cert: fs.readFileSync('./certs/client-cert.pem'),
  ca: fs.readFileSync('./certs/ca-cert.pem'),
  rejectUnauthorized: true
});

async function callMtlsApi() {
  try {
    const { data } = await axios.get('https://mtls-api.example.com/secure', {
      httpsAgent,
      timeout: 10000
    });
    return data;
  } catch (error) {
    if (error.code === 'ETLS_CERT_VERIFY') {
      throw new Error('Client certificate rejected by server');
    }
    throw error;
  }
}

callMtlsApi()
  .then(data => console.log('mTLS success:', data))
  .catch(err => console.error('mTLS error:', err.message));
```

**Expected Output:**
```
mTLS success: { "message": "Authenticated via mTLS" }
```

**Why this output:** The `https.Agent` presents the client's certificate during the TLS handshake. The server validates the client certificate against its trusted CA. If validation succeeds, the request proceeds. If the client certificate is rejected, the TLS handshake fails with an `ETLS_CERT_VERIFY` error.

---

## Core Concept 3: Resiliency Patterns

### Definitions

**Core Definition:** Resiliency patterns are architectural and implementation techniques that enable a system to continue functioning (or degrade gracefully) when external dependencies fail, slow down, or become unavailable.

**Technical Definition:** Resiliency patterns include timeouts (bound the time waiting for a response), retries with exponential backoff (retry failed requests with increasing delays), rate limiting (control the volume of requests), and circuit breakers (stop sending requests to a failing service for a period). These patterns are often combined: a timeout triggers a retry, retries exhaust and trip the circuit breaker, and the circuit breaker returns a fallback response.

**Beginner-Friendly Explanation:** Imagine calling a customer service line. If no one answers after 30 seconds, you hang up (timeout). If you get a busy signal, you wait a bit and try again, waiting longer each time (exponential backoff). If you've tried five times and it's always busy, you stop calling for a while (circuit breaker) and use an alternative (fallback). This is exactly how resilient code handles failing external APIs.

### Purposes

- To prevent one failing service from cascading failures across the entire application.
- To improve user experience by returning fallback responses instead of hanging.
- To reduce load on struggling external services by backing off.
- To detect and isolate failures automatically.

### Sub-Feature 3.1: Timeouts

#### Syntax Rules and Structure

**Axios:**
```js
const response = await axios.get(url, { timeout: 5000 });
```

**Fetch:**
```js
const response = await fetch(url, { signal: AbortSignal.timeout(5000) });
```

| Client | Timeout Mechanism |
|--------|------------------|
| Axios | `timeout` option (milliseconds). |
| Fetch | `AbortSignal.timeout()` or manual `AbortController`. |
| Got | `timeout` option with granular phases. |

**Rules:**
- Always set a timeout on external API calls; never wait indefinitely.
- Use different timeouts for different phases (connect, response, overall).
- Timeout values should be shorter than the circuit breaker's timeout.

---

### Sub-Feature 3.2: Exponential Backoff Retries

#### Definitions

**Core Definition:** Exponential backoff is a retry strategy where the delay between retries increases exponentially (e.g., 1s, 2s, 4s, 8s) to avoid overwhelming a failing service.

**Technical Definition:** The `axios-retry` plugin intercepts failed requests and retries them with an exponentially increasing delay. The delay is calculated as `baseDelay * 2^retryCount`, optionally with jitter to prevent thundering herd.

#### Syntax Rules and Structure

```js
const axiosRetry = require('axios-retry').default;

axiosRetry(axios, {
  retries: 3,
  retryDelay: axiosRetry.exponentialDelay,
  retryCondition: (error) => {
    return axiosRetry.isNetworkOrIdempotentRequestError(error) ||
           error.response?.status === 429;
  }
});
```

| Option | Description |
|--------|-------------|
| `retries` | Maximum number of retry attempts. |
| `retryDelay` | Function returning delay in ms. |
| `retryCondition` | Function determining if a retry should occur. |

#### Annotated Code Example

```js
// exponential-backoff.js
const axios = require('axios');
const axiosRetry = require('axios-retry').default;

// Configure retry with exponential backoff
axiosRetry(axios, {
  retries: 3,
  retryDelay: (retryCount) => {
    const delay = Math.pow(2, retryCount) * 1000; // 2s, 4s, 8s
    return delay + Math.random() * 1000; // Add jitter
  },
  retryCondition: (error) => {
    // Retry on network errors and 5xx/429 responses
    return axiosRetry.isNetworkOrIdempotentRequestError(error) ||
           error.response?.status === 429 ||
           (error.response?.status >= 500 && error.response?.status < 600);
  },
  onRetry: (retryCount, error, requestConfig) => {
    console.log(`Retry ${retryCount} for ${requestConfig.url}: ${error.message}`);
  }
});

async function fetchWithRetry(url) {
  try {
    const { data } = await axios.get(url, { timeout: 5000 });
    return { success: true, data };
  } catch (error) {
    return { success: false, error: error.message };
  }
}

fetchWithRetry('https://httpstat.us/500')
  .then(result => console.log('Result:', result.success));
```

**Expected Output:**
```
Retry 1 for https://httpstat.us/500: Request failed with status code 500
Retry 2 for https://httpstat.us/500: Request failed with status code 500
Retry 3 for https://httpstat.us/500: Request failed with status code 500
Result: false
```

**Why this output:** The first request fails with a 500 status. The retry condition matches (5xx response), so the first retry occurs after ~2 seconds. The second retry occurs after ~4 seconds, and the third after ~8 seconds. After exhausting all retries, the error is returned. Jitter is added to prevent multiple clients from retrying simultaneously.

---

### Sub-Feature 3.3: Rate Limiting (express-rate-limit)

#### Definitions

**Core Definition:** Rate limiting restricts the number of requests a client can make to your API within a time window, protecting your server from abuse and your external dependencies from being overwhelmed.

**Technical Definition:** `express-rate-limit` is middleware that tracks request counts per IP (or custom key) and returns a 429 status when the limit is exceeded. For distributed systems, a Redis store (`rate-limit-redis`) ensures consistent limits across multiple server instances.

#### Syntax Rules and Structure

```js
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

const limiter = rateLimit({
  store: new RedisStore({ client: redisClient }),
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // 100 requests per window
  standardHeaders: true,
  legacyHeaders: false,
  handler: (req, res) => {
    res.status(429).json({
      error: 'Too Many Requests',
      retry_after: Math.ceil(req.rateLimit.resetTime / 1000)
    });
  }
});

app.use('/api/', limiter);
```

#### Annotated Code Example

```js
// rate-limiting.js
const express = require('express');
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const redis = require('redis');
const app = express();

const redisClient = redis.createClient();
redisClient.connect();

// General API limiter
const apiLimiter = rateLimit({
  store: new RedisStore({ client: redisClient, prefix: 'rl:' }),
  windowMs: 15 * 60 * 1000,
  max: 100,
  standardHeaders: true,
  legacyHeaders: false,
  handler: (req, res) => {
    res.status(429).json({
      error: 'Too Many Requests',
      retry_after: Math.ceil(req.rateLimit.resetTime / 1000)
    });
  }
});

// Strict limiter for auth endpoints
const authLimiter = rateLimit({
  store: new RedisStore({ client: redisClient, prefix: 'rl:auth:' }),
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: { error: 'Too many login attempts' }
});

app.use('/api/', apiLimiter);
app.post('/api/auth/login', authLimiter, (req, res) => {
  res.json({ message: 'Login successful' });
});

app.listen(3000, () => console.log('Rate limiting on 3000'));
```

**Expected Output (for the 6th login attempt within 15 minutes):**
```json
{
  "error": "Too many login attempts"
}
```

**Why this output:** The `authLimiter` allows only 5 requests per 15 minutes per IP. The 6th request is rejected with a 429 status and the custom error message. The general API limiter allows 100 requests per 15 minutes. The Redis store ensures limits are shared across all server instances.

---

### Sub-Feature 3.4: Circuit Breaker (opossum)

#### Definitions

**Core Definition:** A circuit breaker is a resilience pattern that monitors failures to a service and, when failures exceed a threshold, "trips" the circuit to prevent further requests, returning a fallback response instead.

**Technical Definition:** The `opossum` library implements the circuit breaker pattern with three states: **Closed** (normal operation), **Open** (requests fail immediately with a fallback), and **Half-Open** (a trial request determines if the service has recovered). It monitors failures over a rolling window and trips when the error percentage exceeds `errorThresholdPercentage`.

#### Syntax Rules and Structure

```js
const CircuitBreaker = require('opossum');

const options = {
  timeout: 3000,                    // Fail if takes > 3s
  errorThresholdPercentage: 50,     // Open after 50% errors
  resetTimeout: 30000,              // Try again after 30s
  rollingCountTimeout: 10000,       // 10s rolling window
  rollingCountBuckets: 10           // 10 buckets of 1s each
};

const breaker = new CircuitBreaker(callExternalAPI, options);
breaker.fallback(() => ({ data: 'fallback' }));
```

#### Annotated Code Example

```js
// circuit-breaker.js
const CircuitBreaker = require('opossum');

// Simulated external API call
async function callExternalAPI() {
  const response = await fetch('https://api.example.com/data', {
    signal: AbortSignal.timeout(5000)
  });
  if (!response.ok) throw new Error('API error');
  return response.json();
}

// Configure circuit breaker
const options = {
  timeout: 3000,
  errorThresholdPercentage: 50,
  resetTimeout: 30000,
  rollingCountTimeout: 10000,
  rollingCountBuckets: 10,
  name: 'api-breaker'
};

const breaker = new CircuitBreaker(callExternalAPI, options);

// Event handlers
breaker.on('open', () => console.log('Circuit opened'));
breaker.on('halfOpen', () => console.log('Circuit half-open'));
breaker.on('close', () => console.log('Circuit closed'));
breaker.on('fallback', (result) => console.log('Fallback:', result));

// Fallback
breaker.fallback(() => ({ data: 'fallback data', source: 'circuit-breaker' }));

// Execute with circuit breaker
async function getData() {
  try {
    const result = await breaker.fire();
    return { success: true, data: result };
  } catch (err) {
    return { success: false, error: err.message };
  }
}

getData().then(result => console.log('Result:', result.success));
```

**Expected Output (when API is down):**
```
Circuit opened
Fallback: { data: 'fallback data', source: 'circuit-breaker' }
Result: false
```

**Why this output:** When the external API fails repeatedly and exceeds the 50% error threshold within the rolling window, the circuit opens. Subsequent requests immediately invoke the fallback function without attempting the failing API call. After the reset timeout (30 seconds), the circuit enters half-open state and allows a trial request to test if the service has recovered.

---

## Core Concept 4: Webhooks

### Definitions

**Core Definition:** A webhook is a push-based mechanism where an external service sends HTTP POST requests to a URL you provide when specific events occur, enabling real-time notifications without polling.

**Technical Definition:** Webhooks deliver event payloads to your server. To ensure authenticity, the sender signs the payload (typically with HMAC-SHA256) and includes the signature in a header (e.g., `Stripe-Signature`, `X-Hub-Signature-256`). Your server must verify the signature using the shared secret before processing the event. Because webhooks can be retried, handlers must be idempotent to prevent duplicate processing.

**Beginner-Friendly Explanation:** A webhook is like a doorbell — when something happens (a payment succeeds, a repository is pushed to), the external service rings your server's doorbell with a message. You need to verify that the ring came from the real service (not an imposter), and you need to handle the same ring multiple times without doing the same thing twice.

### Purposes

- To receive real-time notifications of external events.
- To verify that incoming events are authentic and untampered.
- To handle duplicate deliveries idempotently.
- To process events asynchronously without blocking the sender.

### Sub-Feature 4.1: Stripe Webhook Signature Verification

#### Syntax Rules and Structure

```js
// Stripe requires the raw body for signature verification
app.post('/webhook/stripe',
  express.raw({ type: 'application/json' }),
  (req, res) => {
    const sig = req.headers['stripe-signature'];
    const event = stripe.webhooks.constructEvent(
      req.body,           // Raw body string
      sig,                // Stripe-Signature header
      process.env.STRIPE_WEBHOOK_SECRET
    );
    // Process event
  }
);
```

**Critical rule:** `app.use(express.json())` must be placed **after** the webhook route. If JSON parsing runs first, it mutates the body and signature verification fails.

#### Annotated Code Example

```js
// stripe-webhook.js
const express = require('express');
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const app = express();

// Webhook route MUST come before express.json()
app.post('/webhook/stripe',
  express.raw({ type: 'application/json' }),
  (req, res) => {
    const sig = req.headers['stripe-signature'];

    let event;
    try {
      event = stripe.webhooks.constructEvent(
        req.body,
        sig,
        process.env.STRIPE_WEBHOOK_SECRET
      );
    } catch (err) {
      console.error('Signature verification failed:', err.message);
      return res.status(400).send(`Webhook Error: ${err.message}`);
    }

    // Handle the event
    switch (event.type) {
      case 'payment_intent.succeeded':
        console.log('Payment succeeded:', event.data.object.id);
        break;
      case 'payment_intent.payment_failed':
        console.log('Payment failed:', event.data.object.id);
        break;
      default:
        console.log('Unhandled event type:', event.type);
    }

    res.json({ received: true });
  }
);

// JSON parsing for all other routes
app.use(express.json());

app.listen(3000, () => console.log('Stripe webhook on 3000'));
```

**Expected Output (for a valid webhook event):**
```json
{"received":true}
```

**Expected Output (for an invalid signature):**
```
HTTP/1.1 400 Bad Request
Webhook Error: No signatures found matching the expected signature for payload
```

**Why this output:** `stripe.webhooks.constructEvent()` recomputes the HMAC-SHA256 signature from the raw body and the endpoint secret, then compares it to the `Stripe-Signature` header. If the body was modified (e.g., by JSON parsing middleware), the signature will not match. The raw body parser must be used exclusively for the webhook route.

---

### Sub-Feature 4.2: GitHub Webhook Signature Verification

#### Syntax Rules and Structure

```js
const crypto = require('crypto');

function verifyGitHubSignature(payload, signature, secret) {
  const hmac = crypto.createHmac('sha256', secret);
  const digest = 'sha256=' + hmac.update(payload).digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(digest)
  );
}
```

**Critical rule:** The payload must be the **raw** request body, not `JSON.stringify(req.body)`. If `express.json()` has already parsed the body, `JSON.stringify` will produce different formatting (spacing, key order) and the signature will fail.

#### Annotated Code Example

```js
// github-webhook.js
const express = require('express');
const crypto = require('crypto');
const app = express();

app.post('/webhook/github',
  express.raw({ type: 'application/json' }),
  (req, res) => {
    const signature = req.headers['x-hub-signature-256'];
    const secret = process.env.GITHUB_WEBHOOK_SECRET;

    if (!signature) {
      return res.status(401).send('No signature provided');
    }

    const hmac = crypto.createHmac('sha256', secret);
    const digest = 'sha256=' + hmac.update(req.body).digest('hex');

    const signatureBuffer = Buffer.from(signature);
    const digestBuffer = Buffer.from(digest);

    if (signatureBuffer.length !== digestBuffer.length ||
        !crypto.timingSafeEqual(signatureBuffer, digestBuffer)) {
      return res.status(401).send('Invalid signature');
    }

    // Parse the raw body now that signature is verified
    const event = JSON.parse(req.body.toString());
    const eventType = req.headers['x-github-event'];

    console.log(`Received ${eventType} event`);

    res.json({ received: true });
  }
);

app.use(express.json());
app.listen(3000, () => console.log('GitHub webhook on 3000'));
```

**Expected Output (for a valid webhook):**
```json
{"received":true}
```

**Expected Output (for an invalid signature):**
```
HTTP/1.1 401 Unauthorized
Invalid signature
```

**Why this output:** GitHub signs the raw request body with HMAC-SHA256 using the webhook secret, sending the result in the `X-Hub-Signature-256` header. The server recomputes the HMAC from the raw body and compares it using `timingSafeEqual` (constant-time comparison to prevent timing attacks). If the signatures match, the event is processed.

---

### Sub-Feature 4.3: Idempotency for Webhooks

#### Definitions

**Core Definition:** Idempotency ensures that processing the same webhook event multiple times produces the same result as processing it once, preventing duplicate side effects.

**Technical Definition:** Webhook providers may deliver the same event multiple times (due to network retries or delivery guarantees). An idempotency key (typically the event ID from the provider) is stored in a durable store (Redis, database) with a TTL. Before processing an event, the handler checks if the key has been seen; if so, it returns early without re-processing.

#### Syntax Rules and Structure

```js
const Redis = require('ioredis');
const redis = new Redis();

async function isDuplicate(eventId, ttlSeconds = 86400) {
  const key = `webhook:${eventId}`;
  const set = await redis.set(key, '1', 'EX', ttlSeconds, 'NX');
  return set === null; // null means key already existed
}
```

#### Annotated Code Example

```js
// idempotent-webhook.js
const express = require('express');
const crypto = require('crypto');
const Redis = require('ioredis');
const app = express();

const redis = new Redis();

app.post('/webhook/stripe',
  express.raw({ type: 'application/json' }),
  async (req, res) => {
    const sig = req.headers['stripe-signature'];
    // ... verify signature ...

    const event = JSON.parse(req.body.toString());

    // Idempotency check
    const key = `webhook:stripe:${event.id}`;
    const isDuplicate = await redis.set(key, '1', 'EX', 86400, 'NX') === null;

    if (isDuplicate) {
      console.log('Duplicate event ignored:', event.id);
      return res.json({ received: true, duplicate: true });
    }

    // Process event
    console.log('Processing event:', event.id, event.type);
    // ... business logic ...

    res.json({ received: true, duplicate: false });
  }
);

app.listen(3000, () => console.log('Idempotent webhook on 3000'));
```

**Expected Output (for first delivery):**
```json
{"received":true,"duplicate":false}
```

**Expected Output (for duplicate delivery):**
```json
{"received":true,"duplicate":true}
```

**Why this output:** The `redis.set()` with `NX` (only set if not exists) returns `null` if the key already exists. The first delivery sets the key and processes the event. The second delivery finds the key and returns early without re-processing. The TTL (86400 seconds = 24 hours) ensures keys are cleaned up automatically.

---

## References

- Axios vs Fetch vs Got: HTTP Clients 2026 — https://www.pkgpulse.com/guides/axios-vs-fetch-vs-got-2026
- Axios mTLS Handling — https://github.com/axios/axios/issues/6474
- AWS Signature V4 with Node.js and Neptune — https://docs.aws.amazon.com/neptune/latest/userguide/iam-auth-connecting-sparql.html
- aws-sigv4-sign (npm) — https://www.npmjs.com/package/aws-sigv4-sign
- Opossum Circuit Breaker Style — https://github.com/aj-geddes/useful-ai-prompts/blob/HEAD/skills/circuit-breaker-pattern/references/opossum-style-circuit-breaker-nodejs.md
- Opossum Circuit Breaker Options — https://classic.yarnpkg.com/en/package/opossum
- Stripe Webhook Signature Verification — https://docs.stripe.com/webhooks/signature
- Stripe Webhook Handler with Express — https://docs.stripe.com/webhooks/quickstart?lang=node
- GitHub Webhook Signature Validation — https://github.com/orgs/community/discussions/199399
- express-rate-limit Documentation — https://www.npmjs.com/package/express-rate-limit
- Rate Limiting in Node.js — https://coreui.io/blog/how-to-implement-rate-limiting-in-node-js/
- OWASP REST Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- Node.js Fetch API — https://nodejs.org/api/globals.html#fetch
- Undici Documentation — https://undici.nodejs.org/
- Got Documentation — https://github.com/sindresorhus/got
- axios-retry Documentation — https://github.com/softonic/axios-retry
- ioredis Documentation — https://github.com/redis/ioredis