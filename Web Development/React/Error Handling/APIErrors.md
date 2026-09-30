# API & Data Fetching Errors: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** API & Data Fetching Errors in React refers to the systematic identification, categorisation, and handling of failures that occur when a React application communicates with external servers, APIs, or data sources.

**Technical Definition:** API and data-fetching errors are exceptions or unexpected states that arise during HTTP communication (via `fetch`, Axios, or higher-level libraries like TanStack Query and RTK Query), runtime data validation, authentication/authorisation checks, or asynchronous execution within React components. These errors can be broadly categorised into: network-level failures (DNS resolution, connection timeouts, CORS), HTTP-level failures (4xx client errors, 5xx server errors), validation failures (schema mismatches, malformed payloads), authentication/authorisation failures (401 Unauthorized, 403 Forbidden), and asynchronous execution failures (errors thrown inside `useEffect`, event handlers, or custom hooks). React's architecture provides multiple interception points—`try`/`catch` blocks, `AbortController`, Axios interceptors, Error Boundaries, and library-specific error callbacks—each suited to different failure modes.

**Beginner-Friendly Explanation:** When your React app asks a server for data, things can go wrong. The server might be down, the network might be slow, the data might be in the wrong format, or the user might not have permission to see it. Handling API errors means writing code that anticipates these problems and shows users a helpful message instead of a broken page. It's like having a plan B for every request your app makes.

### Key Characteristics

- **Multi-Layered:** Errors can occur at the network layer, HTTP layer, application layer (validation), and component layer (rendering), requiring different handling strategies at each level.
- **Asynchronous by Nature:** API calls are asynchronous, meaning errors surface via Promises, callbacks, or thrown exceptions in async functions—not synchronously.
- **Context-Dependent:** The appropriate error handling strategy depends on the type of error (400 vs 500 vs 401 vs network failure) and where it occurs (event handler vs `useEffect` vs render).
- **Library-Specific:** Different data-fetching libraries (TanStack Query, RTK Query, Axios) provide different error handling APIs and patterns.
- **User-Facing vs Developer-Facing:** Some errors require user-facing feedback (validation messages, retry buttons), while others are logged for developers (unexpected schema mismatches, 500 errors).
- **Recoverable vs Non-Recoverable:** Some errors can be recovered from (retry, token refresh, re-authentication), while others require navigation away or a full page reload.

### Prerequisites

- Solid understanding of JavaScript Promises and `async`/`await`.
- Familiarity with React Hooks (`useState`, `useEffect`) and component lifecycle.
- Basic knowledge of HTTP status codes (2xx, 4xx, 5xx) and REST API concepts.
- Understanding of `fetch` API or Axios for making HTTP requests.
- Awareness of React Error Boundaries for component-level error handling.

### Related Programming Areas

- **HTTP Protocol:** Status codes, headers, CORS, and network fundamentals.
- **Promise-Based Error Handling:** `try`/`catch`, `.then()`/`.catch()`, and unhandled rejection patterns.
- **Data Validation:** Runtime schema validation with Zod, Yup, or similar libraries.
- **Authentication & Authorisation:** JWT tokens, session management, and role-based access control.
- **Observability:** Error logging, monitoring, and reporting (Sentry, LogRocket).
- **State Management:** Coordinating error state across components (Context, Redux, Zustand).

### Core Concepts / Features

1. Network & HTTP Failures
2. Validation & Schema Errors
3. Auth States (401 & 403)
4. Asynchronous Hook Interceptors

---

## Core Concept 1: Network & HTTP Failures

### Definitions

**Core Definition:** Network and HTTP failures are errors that occur when a request cannot be completed due to connectivity issues, timeouts, or the server returning a non-2xx HTTP status code.

**Technical Definition:** Network failures manifest as `TypeError` exceptions (e.g., "Failed to fetch") when the browser cannot establish a connection to the server due to DNS failure, network disconnection, CORS policy violations, or server unavailability. HTTP failures occur when the server responds with a status code in the 4xx or 5xx range. The `fetch` API does not reject on HTTP errors; it only rejects on network failures. Developers must check `response.ok` (or `response.status`) to detect HTTP errors. Status codes in the 400–499 range indicate client errors (bad request, unauthorised, not found, etc.), while 500–599 indicate server errors (internal error, gateway timeout, etc.). Timeout constraints are not natively supported by `fetch` and must be implemented using `AbortController` with `setTimeout`.

**Beginner-Friendly Explanation:** When you ask a server for data, two things can go wrong. First, you might not reach the server at all—your internet is down, the server is offline, or there's a misconfiguration. Second, you might reach the server but it responds with an error, like "I don't understand your request" (400) or "I'm broken right now" (500). The `fetch` API only throws an error for the first case; for the second case, you have to check the status code yourself.

### Purposes

- To detect and handle network connectivity failures gracefully.
- To distinguish between client errors (4xx) and server errors (5xx) and respond accordingly.
- To implement timeout constraints for requests that take too long.
- To cancel in-flight requests when a component unmounts or dependencies change.
- To provide meaningful feedback to users when a request fails.
- To enable retry logic for transient server errors.

### Syntax Rules and Structure

**General Syntax for `fetch` Error Handling:**
```jsx
async function fetchData(url) {
  try {
    const response = await fetch(url);

    // fetch() only rejects on network errors, NOT on HTTP errors
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }

    const data = await response.json();
    return data;
  } catch (error) {
    // Handles both network errors and our thrown HTTP errors
    console.error('Fetch failed:', error);
    throw error;
  }
}
```

**Component Breakdown:**
- `await fetch(url)`: Initiates the request. If the network fails, this throws a `TypeError`.
- `if (!response.ok)`: Checks whether the status code is in the 200–299 range. If not, an error is thrown manually.
- `response.status`: The HTTP status code (e.g., 400, 404, 500).
- `catch (error)`: Catches both network errors (from `fetch`) and HTTP errors (from the manual throw).

**General Syntax for Timeout with `AbortController`:**
```jsx
async function fetchWithTimeout(url, timeoutMs = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs);

  try {
    const response = await fetch(url, { signal: controller.signal });
    clearTimeout(timeoutId);

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    clearTimeout(timeoutId);

    // Distinguish timeout from other errors
    if (error.name === 'AbortError') {
      throw new Error(`Request timed out after ${timeoutMs}ms`);
    }

    throw error;
  }
}
```

**Component Breakdown:**
- `new AbortController()`: Creates a controller for cancelling the request.
- `setTimeout(() => controller.abort(), timeoutMs)`: Schedules an abort after the timeout.
- `{ signal: controller.signal }`: Passes the abort signal to `fetch`.
- `error.name === 'AbortError'`: Detects whether the error was caused by the abort (timeout).
- `clearTimeout(timeoutId)`: Clears the timeout if the request completes or fails before the timeout.

**General Syntax for Axios Error Handling:**
```jsx
import axios from 'axios';

async function fetchData(url) {
  try {
    const response = await axios.get(url);
    return response.data;
  } catch (error) {
    if (error.response) {
      // Server responded with a status code outside 2xx
      console.error('HTTP error:', error.response.status, error.response.data);
    } else if (error.request) {
      // Request was made but no response received (network error)
      console.error('Network error:', error.message);
    } else {
      // Something else happened (e.g., request configuration error)
      console.error('Request setup error:', error.message);
    }
    throw error;
  }
}
```

**Component Breakdown:**
- `error.response`: Present when the server responded with a non-2xx status.
- `error.response.status`: The HTTP status code.
- `error.request`: Present when the request was made but no response was received.
- `error.message`: A descriptive error message.
- Axios rejects on both network and HTTP errors, unlike `fetch`.

**Syntax Rules:**
- Always check `response.ok` when using `fetch`; it does not reject on HTTP errors.
- Use `AbortController` for timeout constraints; `fetch` has no built-in timeout.
- Distinguish between network errors (`TypeError` or `error.request`) and HTTP errors (`response.status`).
- Clear timeout timers in a `finally` block or after the request completes to avoid memory leaks.
- For Axios, use `error.response` to detect HTTP errors and `error.request` for network errors.
- Client errors (4xx) generally should not be retried; server errors (5xx) may be retried with backoff.

**Constraints and Limitations:**
- `fetch` does not support timeout natively; `AbortController` must be used.
- `AbortController` is single-use; create a new one for each request.
- CORS errors appear as network errors and cannot be distinguished from other network failures in JavaScript.
- `AbortSignal.timeout()` is available in modern browsers but not in older environments.
- Axios automatically parses JSON; `fetch` requires `response.json()`.
- Some HTTP status codes (e.g., 401) require special handling beyond generic error display.

### Annotated Code Examples

**Example 1: Comprehensive Fetch with Error Classification**

```jsx
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchUser() {
      try {
        setLoading(true);
        setError(null);

        const response = await fetch(
          `https://jsonplaceholder.typicode.com/users/${userId}`,
          { signal: controller.signal }
        );

        // Classify HTTP errors by status code
        if (!response.ok) {
          if (response.status >= 500) {
            throw new Error('Server error. Please try again later.');
          } 
          else if (response.status === 404) {
            throw new Error('User not found.');
          } 
          else if (response.status === 400) {
            throw new Error('Invalid request. Please check your input.');
          } 
          else {
            throw new Error(`Request failed (${response.status}).`);
          }
        }

        const data = await response.json();
        setUser(data);
      } catch (err) {
        // Ignore abort errors (expected during cleanup)
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchUser();

    return () => controller.abort();
  }, [userId]);

  if (loading) return <p>Loading user profile...</p>;
  if (error) return <p role="alert" style={{ color: 'red' }}>{error}</p>;
  if (!user) return <p>No user data available.</p>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
      <p>Phone: {user.phone}</p>
    </div>
  );
}

export default UserProfile;
```

**Expected Output:** The component displays "Loading user profile..." initially. If the user is found, their name, email, and phone are displayed. If the server returns a 404, "User not found." is displayed. If the server returns a 500, "Server error. Please try again later." is displayed. Network errors display the browser's error message.

**Why This Output Occurs:** The `fetch` call is wrapped in a `try`/`catch` block. The `response.ok` check classifies HTTP errors by status code, throwing different error messages for 4xx and 5xx errors. The `catch` block captures both network errors (from `fetch` itself) and the manually thrown HTTP errors. The `AbortController` ensures the request is cancelled if the component unmounts or `userId` changes.

**Example 2: Axios with Timeout and Error Classification**

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

// Create an Axios instance with a default timeout
const apiClient = axios.create({
  timeout: 5000, // 5-second timeout
  headers: { 'Content-Type': 'application/json' },
});

function PostList() {
  const [posts, setPosts] = useState([]);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchPosts() {
      try {
        setLoading(true);
        setError(null);

        const response = await apiClient.get(
          'https://jsonplaceholder.typicode.com/posts',
          { signal: controller.signal }
        );

        setPosts(response.data.slice(0, 5));
      } catch (err) {
        if (axios.isCancel(err)) {
          // Request was cancelled; ignore
          return;
        }

        if (err.code === 'ECONNABORTED' && err.message.includes('timeout')) {
          setError('Request timed out. Please check your connection.');
        } else if (err.response) {
          // Server responded with non-2xx status
          if (err.response.status >= 500) {
            setError('Server error. Please try again later.');
          } else if (err.response.status === 404) {
            setError('Resource not found.');
          } else {
            setError(`Request failed: ${err.response.status}`);
          }
        } else if (err.request) {
          // Request made but no response received
          setError('Network error. Please check your connection.');
        } else {
          setError('An unexpected error occurred.');
        }
      } finally {
        setLoading(false);
      }
    }

    fetchPosts();

    return () => controller.abort();
  }, []);

  if (loading) return <p>Loading posts...</p>;
  if (error) return <p role="alert" style={{ color: 'red' }}>{error}</p>;

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

export default PostList;
```

**Expected Output:** The component displays "Loading posts..." initially. If successful, the first five post titles are displayed. If the request times out, "Request timed out. Please check your connection." is displayed. If the server returns a 500, "Server error. Please try again later." is displayed. Network errors display "Network error. Please check your connection."

**Why This Output Occurs:** Axios provides a rich error object: `err.response` for HTTP errors, `err.request` for network errors, and `err.code === 'ECONNABORTED'` for timeouts. The `axios.isCancel(err)` check distinguishes cancellations from actual errors. The `timeout: 5000` configuration ensures requests are automatically aborted after 5 seconds. The `AbortController` provides an additional layer of cancellation for component unmounting.

### Real-World Cases

- **E-commerce checkout:** Handling payment gateway timeouts with a "Try Again" button.
- **Dashboard data loading:** Displaying "Server error" with a retry option for 500 errors.
- **Search results:** Showing "No results found" for 404 errors and "Invalid search" for 400 errors.
- **User profile pages:** Showing "User not found" for 404 errors instead of a generic error.
- **Real-time feeds:** Using `AbortController` to cancel stale requests when the user navigates away.
- **Mobile applications:** Detecting network disconnection and showing "No internet connection."

---

## Core Concept 2: Validation & Schema Errors

### Definitions

**Core Definition:** Validation and schema errors are failures that occur when API responses or form inputs do not match the expected data structure, type, or format.

**Technical Definition:** Runtime validation errors occur when an API returns data that does not conform to the expected schema—missing required fields, incorrect types, malformed structures, or unexpected values. TypeScript's static types are erased at runtime, so they cannot catch these mismatches. Libraries like Zod and Yup provide runtime schema validation that throws descriptive errors (or returns result objects) when data does not match the schema. Zod's `parse()` method throws a `ZodError` on validation failure, while `safeParse()` returns a result object with a `success` boolean and an `error` property. Yup's `validate()` method returns a Promise that rejects with a `ValidationError` on failure. These validation errors must be caught and handled separately from network/HTTP errors.

**Beginner-Friendly Explanation:** Imagine you order a pizza and the delivery person brings you a burger instead. Your app has the same problem: it expects data in a certain shape (like a user with a name and email), but the API might send something completely different. Validation libraries like Zod and Yup let you define exactly what the data should look like, and they'll tell you if the API sends something wrong—before your app crashes trying to use it.

### Purposes

- To detect mismatches between expected and actual API response structures.
- To prevent runtime crashes caused by accessing undefined properties.
- To provide descriptive error messages when data is malformed.
- To validate form inputs before submission.
- To create a single source of truth for data shapes (runtime validation + TypeScript types).
- To handle server-side validation errors and display them inline in forms.

### Syntax Rules and Structure

**General Syntax with Zod:**
```jsx
import { z } from 'zod';

// Define the schema
const UserSchema = z.object({
  id: z.string(),
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Invalid email format'),
  age: z.number().int().positive().optional(),
});

// Infer the TypeScript type from the schema
type User = z.infer<typeof UserSchema>;

// Validate API response
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();

  // parse() throws a ZodError on failure
  return UserSchema.parse(data);
}

// Alternative: safeParse() returns a result object
async function fetchUserSafely(id) {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();

  const result = UserSchema.safeParse(data);

  if (!result.success) {
    console.error('Validation failed:', result.error.issues);
    return null;
  }

  return result.data;
}
```

**Component Breakdown:**
- `z.object({ ... })`: Defines an object schema with typed fields.
- `z.string().email()`: Validates that the value is a valid email string.
- `z.infer<typeof UserSchema>`: Extracts the TypeScript type from the schema.
- `UserSchema.parse(data)`: Throws a `ZodError` if validation fails.
- `UserSchema.safeParse(data)`: Returns `{ success: true, data }` or `{ success: false, error }`.

**General Syntax with Yup and React Hook Form:**
```jsx
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

// Define the schema
const schema = yup.object({
  name: yup.string().required('Name is required'),
  email: yup.string().email('Invalid email').required('Email is required'),
  age: yup.number().positive().integer().min(18, 'Must be at least 18'),
}).required();

function RegistrationForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm({
    resolver: yupResolver(schema),
  });

  const onSubmit = (data) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} placeholder="Name" />
      {errors.name && <p>{errors.name.message}</p>}

      <input {...register('email')} placeholder="Email" />
      {errors.email && <p>{errors.email.message}</p>}

      <input {...register('age')} placeholder="Age" type="number" />
      {errors.age && <p>{errors.age.message}</p>}

      <button type="submit">Register</button>
    </form>
  );
}
```

**Component Breakdown:**
- `yup.object({ ... })`: Defines the validation schema.
- `yup.string().required('Name is required')`: Validates that the field is a non-empty string.
- `yupResolver(schema)`: Integrates Yup with React Hook Form.
- `errors.name`: Contains validation errors for the `name` field.
- `handleSubmit(onSubmit)`: Only calls `onSubmit` if validation passes.

**Syntax Rules:**
- Define schemas outside the component to avoid re-creating them on every render.
- Use `parse()` when you want validation errors to throw (caught by Error Boundaries or `try`/`catch`).
- Use `safeParse()` when you want to handle validation errors gracefully without exceptions.
- With React Hook Form, use `yupResolver` or `zodResolver` to integrate schema validation.
- Validation errors should be displayed inline for form fields; API response validation errors should be logged and optionally shown as a generic error.
- Always validate on both client and server; never trust client-side validation alone.

**Constraints and Limitations:**
- Zod and Yup add bundle size; consider the trade-off for small applications.
- `parse()` throws, which means you need `try`/`catch` or an Error Boundary to handle it.
- `safeParse()` does not throw, but you must handle the `result.success` check manually.
- Schema validation adds a runtime performance cost; for very large responses, consider sampling or lazy validation.
- Zod is TypeScript-first; Yup is more widely used with Formik.
- Validation errors from the server (e.g., 422 Unprocessable Entity) need to be mapped to form fields manually.

### Annotated Code Examples

**Example 1: Zod Validation for API Response**

```jsx
import React, { useState, useEffect } from 'react';
import { z } from 'zod';

// Define the expected schema for a post
const PostSchema = z.object({
  id: z.number(),
  title: z.string().min(1),
  body: z.string().min(1),
  userId: z.number().int().positive(),
});

const PostArraySchema = z.array(PostSchema);

function PostList() {
  const [posts, setPosts] = useState([]);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchPosts() {
      try {
        setLoading(true);
        setError(null);

        const response = await fetch(
          'https://jsonplaceholder.typicode.com/posts'
        );

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }

        const rawData = await response.json();

        // Runtime validation with Zod
        const result = PostArraySchema.safeParse(rawData);

        if (!result.success) {
          // Log detailed validation issues
          console.error('Validation errors:', result.error.issues);
          setError('Received malformed data from server.');
          return;
        }

        setPosts(result.data.slice(0, 5));
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchPosts();
  }, []);

  if (loading) return <p>Loading posts...</p>;
  if (error) return <p role="alert">{error}</p>;

  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>
          <strong>{post.title}</strong>
          <p>{post.body.substring(0, 50)}...</p>
        </li>
      ))}
    </ul>
  );
}

export default PostList;
```

**Expected Output:** The component displays "Loading posts..." initially. If the API returns valid data, the first five posts are displayed. If the API returns data that doesn't match the schema (e.g., a missing `title` field), "Received malformed data from server." is displayed, and the validation issues are logged to the console.

**Why This Output Occurs:** `PostArraySchema.safeParse(rawData)` validates the entire array of posts against the schema. If any post fails validation, `result.success` is `false`, and the component sets an error message. The `result.error.issues` array contains detailed information about each validation failure, which is logged for debugging. This prevents the component from crashing when it tries to access `post.title` on a malformed object.

**Example 2: Yup Validation with React Hook Form and Server Errors**

```jsx
import React, { useState } from 'react';
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

const schema = yup.object({
  username: yup.string().required('Username is required').min(3, 'At least 3 characters'),
  email: yup.string().email('Invalid email').required('Email is required'),
  password: yup.string().required('Password is required').min(8, 'At least 8 characters'),
}).required();

function RegistrationForm() {
  const [serverError, setServerError] = useState(null);
  const [success, setSuccess] = useState(false);

  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    setError,
    reset,
  } = useForm({
    resolver: yupResolver(schema),
  });

  const onSubmit = async (data) => {
    setServerError(null);
    setSuccess(false);

    try {
      const response = await fetch('/api/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });

      if (!response.ok) {
        const errorData = await response.json();

        // Map server validation errors to form fields
        if (response.status === 422 && errorData.errors) {
          Object.entries(errorData.errors).forEach(([field, message]) => {
            setError(field, { type: 'server', message });
          });
          return;
        }

        throw new Error(errorData.message || 'Registration failed');
      }

      setSuccess(true);
      reset();
    } catch (err) {
      setServerError(err.message);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register('username')} placeholder="Username" />
        {errors.username && <p style={{ color: 'red' }}>{errors.username.message}</p>}
      </div>

      <div>
        <input {...register('email')} placeholder="Email" />
        {errors.email && <p style={{ color: 'red' }}>{errors.email.message}</p>}
      </div>

      <div>
        <input {...register('password')} type="password" placeholder="Password" />
        {errors.password && <p style={{ color: 'red' }}>{errors.password.message}</p>}
      </div>

      {serverError && <p style={{ color: 'red' }}>{serverError}</p>}
      {success && <p style={{ color: 'green' }}>Registration successful!</p>}

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Registering...' : 'Register'}
      </button>
    </form>
  );
}

export default RegistrationForm;
```

**Expected Output:** The form displays three fields (username, email, password) with validation error messages beneath each field when they fail client-side validation. If the server returns a 422 with field-specific errors, those are displayed inline. If the server returns a 500, "Registration failed" appears at the bottom. On success, "Registration successful!" appears, and the form is reset.

**Why This Output Occurs:** The `yupResolver(schema)` integrates Yup validation with React Hook Form, automatically populating the `errors` object. The `setError` function maps server-side validation errors (from a 422 response) to specific form fields, ensuring they appear inline. The `isSubmitting` state disables the submit button during the request. The `serverError` state handles non-field-specific errors.

### Real-World Cases

- **User registration:** Validating form inputs with Yup and displaying inline errors.
- **API response validation:** Using Zod to validate user data before rendering, preventing crashes from malformed responses.
- **E-commerce checkout:** Validating shipping and payment data on both client and server.
- **Content management:** Validating article data from a CMS before rendering.
- **Financial applications:** Validating transaction data to prevent display errors from unexpected values.
- **Multi-step forms:** Validating each step with schema before allowing progression.

---

## Core Concept 3: Auth States (401 & 403)

### Definitions

**Core Definition:** Auth state errors are HTTP 401 (Unauthorized) and 403 (Forbidden) responses that indicate the user is either not authenticated or not authorised to access a resource.

**Technical Definition:** HTTP 401 Unauthorized indicates that the request lacks valid authentication credentials—the user is not logged in, or their token has expired. The correct response is to redirect the user to a login page or attempt a token refresh. HTTP 403 Forbidden indicates that the user is authenticated but does not have permission to access the resource. The correct response is to keep the user on the current page and display a "permission denied" message, because re-authenticating will not grant access. Conflating these two states is a common mistake that leads to broken navigation and confused users.

**Beginner-Friendly Explanation:** Imagine you're trying to enter a building. A 401 is like not having a key card at all—you need to go get one (log in). A 403 is like having a key card that doesn't work for that particular door—you're already in the building, but you don't have permission for that room. You shouldn't be sent back to the entrance; you should just be told "you can't go in there."

### Purposes

- To distinguish between "not logged in" (401) and "logged in but no permission" (403).
- To trigger automatic token refresh when a 401 occurs.
- To redirect unauthenticated users to the login page while preserving their intended destination.
- To display restricted UI blocks for 403 errors without forcing a redirect.
- To prevent repeated failed requests by handling auth errors centrally.
- To provide a smooth re-authentication experience.

### Syntax Rules and Structure

**General Syntax with Axios Interceptors:**
```jsx
import axios from 'axios';

const apiClient = axios.create({
  baseURL: '/api',
  timeout: 10000,
});

// Request interceptor: attach token
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('accessToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor: handle 401 and 403
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        const refreshToken = localStorage.getItem('refreshToken');
        const { data } = await axios.post('/api/auth/refresh', {
          refreshToken,
        });

        localStorage.setItem('accessToken', data.accessToken);
        originalRequest.headers.Authorization = `Bearer ${data.accessToken}`;

        return apiClient(originalRequest);
      } catch (refreshError) {
        localStorage.clear();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }

    if (error.response?.status === 403) {
      // Do NOT redirect; show permission denied UI
      console.warn('Access forbidden:', originalRequest.url);
    }

    return Promise.reject(error);
  }
);

export default apiClient;
```

**Component Breakdown:**
- `apiClient.interceptors.request.use(...)`: Attaches the access token to every request.
- `apiClient.interceptors.response.use(...)`: Handles responses and errors globally.
- `error.response?.status === 401`: Checks for authentication failure.
- `originalRequest._retry`: Prevents infinite loops if the refreshed token also fails.
- `axios.post('/api/auth/refresh', ...)`: Attempts to refresh the access token.
- `window.location.href = '/login'`: Redirects to login if refresh fails.
- `error.response?.status === 403`: Handles authorisation failure without redirect.

**General Syntax for Protected Routes:**
```jsx
import { Navigate, useLocation } from 'react-router-dom';

function ProtectedRoute({ children, requiredRole }) {
  const location = useLocation();
  const { user, isAuthenticated } = useAuth();

  // 401: Not authenticated — redirect to login, preserve destination
  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  // 403: Authenticated but insufficient role — show restricted UI
  if (requiredRole && user.role !== requiredRole) {
    return (
      <div role="alert">
        <h2>Access Denied</h2>
        <p>You don't have permission to view this page.</p>
      </div>
    );
  }

  return children;
}
```

**Component Breakdown:**
- `isAuthenticated`: Checked first; if false, redirect to login.
- `state={{ from: location }}`: Preserves the intended destination for post-login redirect.
- `requiredRole`: If the user's role doesn't match, show a restricted UI instead of redirecting.
- `children`: Rendered only if the user is authenticated and authorised.

**Syntax Rules:**
- 401 errors should trigger a redirect to login (or token refresh) while preserving the intended URL.
- 403 errors should not redirect; display a permission-denied message or restricted UI.
- Use Axios response interceptors to handle auth errors centrally.
- Guard against infinite loops by marking retried requests (`_retry` flag).
- Clear tokens and redirect to login if token refresh fails.
- Preserve the intended destination so users return to their original page after logging in.

**Constraints and Limitations:**
- Token refresh endpoints must not themselves return 401, or infinite loops may occur.
- Multiple simultaneous 401s can trigger multiple refresh requests; use a queue or lock.
- `window.location.href` causes a full page reload; use React Router's `navigate` for SPA navigation.
- 403 errors are not always about roles; they can also indicate resource-level permissions.
- Server-side rendering (SSR) cannot access `localStorage`; use cookies or handle auth differently.

### Annotated Code Examples

**Example 1: Axios Interceptor with Token Refresh Queue**

```jsx
import axios from 'axios';

const apiClient = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
});

// Queue to hold requests while token is being refreshed
let isRefreshing = false;
let failedQueue = [];

function processQueue(error, token = null) {
  failedQueue.forEach(({ resolve, reject }) => {
    if (error) {
      reject(error);
    } else {
      resolve(token);
    }
  });
  failedQueue = [];
}

apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    // Handle 401 with token refresh
    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
        // Queue the request until refresh completes
        return new Promise((resolve, reject) => {
          failedQueue.push({ resolve, reject });
        }).then((token) => {
          originalRequest.headers.Authorization = `Bearer ${token}`;
          return apiClient(originalRequest);
        });
      }

      originalRequest._retry = true;
      isRefreshing = true;

      try {
        const refreshToken = localStorage.getItem('refreshToken');
        const { data } = await axios.post(
          'https://api.example.com/auth/refresh',
          { refreshToken }
        );

        localStorage.setItem('accessToken', data.accessToken);
        processQueue(null, data.accessToken);

        originalRequest.headers.Authorization = `Bearer ${data.accessToken}`;
        return apiClient(originalRequest);
      } catch (refreshError) {
        processQueue(refreshError);
        localStorage.clear();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      } finally {
        isRefreshing = false;
      }
    }

    // Handle 403 without redirect
    if (error.response?.status === 403) {
      console.warn('Access forbidden:', originalRequest.url);
    }

    return Promise.reject(error);
  }
);

export default apiClient;
```

**Expected Output:** When the access token expires, a 401 response triggers the interceptor. The first 401 starts a token refresh; subsequent 401s are queued. Once the refresh completes, all queued requests are retried with the new token. If the refresh fails, all queued requests are rejected, tokens are cleared, and the user is redirected to `/login`. A 403 response logs a warning but does not redirect.

**Why This Output Occurs:** The `isRefreshing` flag ensures only one refresh request is made at a time. The `failedQueue` holds requests that arrived while the refresh was in progress. The `processQueue` function resolves or rejects all queued requests after the refresh completes or fails. The `_retry` flag prevents infinite loops if the refreshed token also fails. The 403 handler distinguishes authorisation failure from authentication failure.

**Example 2: 401 vs 403 in Component Code**

```jsx
import React, { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import apiClient from './apiClient';

function AdminPanel() {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);
  const navigate = useNavigate();

  useEffect(() => {
    async function fetchAdminData() {
      try {
        setLoading(true);
        setError(null);

        const response = await apiClient.get('/admin/dashboard');
        setData(response.data);
      } catch (err) {
        if (err.response?.status === 401) {
          // 401: Not authenticated — redirect to login
          navigate('/login', { state: { from: '/admin' } });
        } else if (err.response?.status === 403) {
          // 403: Authenticated but not authorised
          setError('You do not have permission to access the admin panel.');
        } else {
          setError('Failed to load admin data. Please try again.');
        }
      } finally {
        setLoading(false);
      }
    }

    fetchAdminData();
  }, [navigate]);

  if (loading) return <p>Loading admin data...</p>;

  if (error) {
    return (
      <div role="alert" style={{ padding: '20px', textAlign: 'center' }}>
        <h2>Access Denied</h2>
        <p>{error}</p>
        <button onClick={() => navigate('/')}>Return to Home</button>
      </div>
    );
  }

  return (
    <div>
      <h1>Admin Dashboard</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
}

export default AdminPanel;
```

**Expected Output:** If the user is not authenticated, they are redirected to `/login`. If the user is authenticated but lacks the admin role, "Access Denied" with "You do not have permission to access the admin panel." is displayed, along with a "Return to Home" button. The user is not redirected to login because re-authenticating would not grant access.

**Why This Output Occurs:** The `err.response?.status` check distinguishes between 401 and 403. For 401, `navigate('/login')` is called with the intended destination preserved. For 403, the error state is set, and a restricted UI is displayed. This respects the semantic difference: 401 means "log in," 403 means "you're logged in but can't go here."

### Real-World Cases

- **Admin dashboards:** Redirecting unauthenticated users to login, showing "Access Denied" for insufficient roles.
- **E-commerce:** Redirecting expired sessions to login while preserving the cart; showing "This product is not available in your region" for 403.
- **SaaS applications:** Refreshing tokens automatically and queuing requests during the refresh.
- **Content platforms:** Showing "Subscribe to view this content" for 403 errors.
- **Multi-tenant applications:** Showing "You don't have access to this organisation" for 403.
- **Banking applications:** Immediately logging out on 401 and redirecting to login for security.

---

## Core Concept 4: Asynchronous Hook Interceptors

### Definitions

**Core Definition:** Asynchronous Hook Interceptors are the patterns and mechanisms for handling errors within `useEffect`, event handlers, and custom data-fetching hooks (such as TanStack Query's `useQuery` and RTK Query's `useGetQuery`).

**Technical Definition:** React's `useEffect` Hook does not support `async` functions directly, so asynchronous operations must be wrapped in an inner async function with `try`/`catch` blocks. Errors in event handlers are not caught by Error Boundaries; they must be handled with `try`/`catch` or by propagating them to a state variable that triggers an Error Boundary. TanStack Query provides built-in error handling through the `error` property returned by `useQuery`, automatic retries with exponential backoff, and integration with Error Boundaries via the `useErrorBoundary` option and `QueryErrorResetBoundary`. RTK Query returns errors in the `error` property of its hooks and provides an `unwrap()` method for accessing the raw Promise result, allowing `try`/`catch` handling at the call site.

**Beginner-Friendly Explanation:** Different parts of your React app handle errors differently. A `useEffect` can't directly use `async`/`await`, so you need a special pattern. Event handlers don't get caught by Error Boundaries, so you need `try`/`catch`. Libraries like TanStack Query and RTK Query have their own error handling systems that make things easier—they track error states, retry automatically, and integrate with Error Boundaries.

### Purposes

- To handle errors that occur during asynchronous operations in `useEffect`.
- To catch and handle errors in event handlers, which are not covered by Error Boundaries.
- To leverage library-specific error handling in TanStack Query and RTK Query.
- To provide consistent error states (`isError`, `error`) across data-fetching hooks.
- To enable retry logic and error recovery for failed queries.
- To integrate asynchronous errors with Error Boundaries for centralised handling.

### Syntax Rules and Structure

**General Syntax for `useEffect` with Async/Catch:**
```jsx
useEffect(() => {
  let ignore = false;

  async function fetchData() {
    try {
      const response = await fetch('/api/data');
      const data = await response.json();
      if (!ignore) setData(data);
    } catch (error) {
      if (!ignore) setError(error);
    }
  }

  fetchData();

  return () => {
    ignore = true;
  };
}, []);
```

**Component Breakdown:**
- `let ignore = false`: A flag to prevent state updates after cleanup.
- `async function fetchData()`: An inner async function, because `useEffect` cannot be async.
- `try`/`catch`: Catches network and HTTP errors.
- `if (!ignore)`: Guards against state updates after unmount.
- `return () => { ignore = true }`: Cleanup function sets the flag.

**General Syntax for TanStack Query:**
```jsx
import { useQuery } from '@tanstack/react-query';

function UserList() {
  const {
    data,
    error,
    isError,
    isLoading,
    isFetching,
    refetch,
  } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
    retry: 2, // Retry failed requests twice
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    throwOnError: (error) => error.status >= 500,
  });

  if (isLoading) return <p>Loading...</p>;
  if (isError) {
    return (
      <div>
        <p>Error: {error.message}</p>
        <button onClick={() => refetch()}>Retry</button>
      </div>
    );
  }

  return (
    <ul>
      {data.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

**Component Breakdown:**
- `useQuery({ queryKey, queryFn })`: The core Hook for data fetching.
- `error`: The error object if the query failed.
- `isError`: Boolean indicating whether the query is in an error state.
- `retry: 2`: Retries the query twice on failure.
- `retryDelay`: Exponential backoff between retries.
- `throwOnError`: When `true`, errors are thrown to the nearest Error Boundary.
- `refetch()`: Manually re-runs the query.

**General Syntax for RTK Query:**
```jsx
import { useGetPostsQuery } from './apiSlice';

function PostList() {
  const {
    data,
    error,
    isLoading,
    isError,
    refetch,
  } = useGetPostsQuery();

  if (isLoading) return <p>Loading...</p>;
  if (isError) {
    return (
      <div>
        <p>Error: {error.status} — {error.data?.message || 'Unknown error'}</p>
        <button onClick={() => refetch()}>Retry</button>
      </div>
    );
  }

  return (
    <ul>
      {data.map(post => <li key={post.id}>{post.title}</li>)}
    </ul>
  );
}
```

**Component Breakdown:**
- `useGetPostsQuery()`: The auto-generated Hook for the endpoint.
- `error.status`: The HTTP status code (e.g., 500, 404).
- `error.data`: The error payload from the server.
- `refetch()`: Manually re-runs the query.

**RTK Query with `unwrap()` for Mutation Error Handling:**
```jsx
function AddPostForm() {
  const [addPost, { isLoading }] = useAddPostMutation();

  async function handleSubmit(formData) {
    try {
      const result = await addPost(formData).unwrap();
      console.log('Post created:', result);
    } catch (error) {
      console.error('Failed to create post:', error.status, error.data);
    }
  }

  // ...
}
```

**Component Breakdown:**
- `addPost(formData)`: Triggers the mutation.
- `.unwrap()`: Returns a standard Promise that resolves with the data or rejects with the error.
- `try`/`catch`: Catches the rejection and handles the error.

**Syntax Rules:**
- Never pass an `async` function directly to `useEffect`; define an inner async function and call it.
- Use a flag or `AbortController` to prevent state updates after unmount.
- In TanStack Query, errors are stored in the `error` property; use `isError` to check.
- In RTK Query, errors are stored in the `error` property of the hook's return value.
- Use `unwrap()` in RTK Query to handle mutation errors with `try`/`catch` at the call site.
- For TanStack Query, use `throwOnError` to propagate errors to Error Boundaries.
- Event handlers must use `try`/`catch` or set error state; they are not caught by Error Boundaries.

**Constraints and Limitations:**
- `useEffect` cannot be `async`; you must use an inner async function.
- Errors in event handlers are not caught by Error Boundaries.
- TanStack Query's default retry count is 3; set `retry: false` to disable.
- RTK Query's `error` object shape depends on the `baseQuery` used.
- TanStack Query's `throwOnError` is a function or boolean; when `true`, all errors go to the Error Boundary.
- `unwrap()` is only available on mutation triggers, not query hooks.

### Annotated Code Examples

**Example 1: `useEffect` with Proper Async Error Handling**

```jsx
import React, { useState, useEffect } from 'react';

function CommentsSection({ postId }) {
  const [comments, setComments] = useState([]);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // AbortController for cancellation
    const controller = new AbortController();

    // Inner async function (useEffect cannot be async)
    async function fetchComments() {
      try {
        setLoading(true);
        setError(null);

        const response = await fetch(
          `https://jsonplaceholder.typicode.com/posts/${postId}/comments`,
          { signal: controller.signal }
        );

        if (!response.ok) {
          if (response.status >= 500) {
            throw new Error('Server error. Please try again later.');
          }
          throw new Error(`Failed to fetch comments (${response.status})`);
        }

        const data = await response.json();
        setComments(data);
      } catch (err) {
        // Ignore abort errors
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchComments();

    // Cleanup: abort the request
    return () => controller.abort();
  }, [postId]);

  if (loading) return <p>Loading comments...</p>;
  if (error) return <p role="alert">{error}</p>;
  if (comments.length === 0) return <p>No comments yet.</p>;

  return (
    <ul>
      {comments.map(comment => (
        <li key={comment.id}>
          <strong>{comment.email}</strong>
          <p>{comment.body}</p>
        </li>
      ))}
    </ul>
  );
}

export default CommentsSection;
```

**Expected Output:** The component displays "Loading comments..." initially. If successful, the comments are displayed. If the server returns a 500, "Server error. Please try again later." is displayed. If the request is aborted (e.g., the user navigates away), no error is displayed. If there are no comments, "No comments yet." is displayed.

**Why This Output Occurs:** The `useEffect` defines an inner `async function fetchComments()` because `useEffect` cannot be `async`. The `try`/`catch` block handles both network and HTTP errors. The `AbortController` cancels the request if the component unmounts or `postId` changes. The `err.name !== 'AbortError'` check prevents abort errors from being displayed as user-facing errors.

**Example 2: TanStack Query with Retry and Error Boundary Integration**

```jsx
import React from 'react';
import { useQuery } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

async function fetchTodos() {
  const response = await fetch('https://jsonplaceholder.typicode.com/todos');
  if (!response.ok) {
    const error = new Error('Failed to fetch todos');
    error.status = response.status;
    throw error;
  }
  return response.json();
}

function TodoList() {
  const { data, isError, error, refetch } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    retry: 2,
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 10000),
    throwOnError: (error) => error.status >= 500,
  });

  if (isError && error.status < 500) {
    return (
      <div role="alert">
        <p>Error: {error.message}</p>
        <button onClick={() => refetch()}>Try Again</button>
      </div>
    );
  }

  return (
    <ul>
      {data?.slice(0, 10).map(todo => (
        <li key={todo.id} style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
          {todo.title}
        </li>
      ))}
    </ul>
  );
}

function App() {
  return (
    <ErrorBoundary
      fallbackRender={({ error, resetErrorBoundary }) => (
        <div role="alert">
          <h2>Something went wrong</h2>
          <p>{error.message}</p>
          <button onClick={resetErrorBoundary}>Try again</button>
        </div>
      )}
    >
      <h1>Todos</h1>
      <TodoList />
    </ErrorBoundary>
  );
}

export default App;
```

**Expected Output:** If the API returns a 500 error, `throwOnError` propagates it to the Error Boundary, which displays "Something went wrong" with the error message and a "Try again" button. If the API returns a 404, the component's own error UI displays "Error: Failed to fetch todos" with a "Try Again" button. The query retries twice before giving up.

**Why This Output Occurs:** The `throwOnError` option is a function that returns `true` for 5xx errors, causing them to be thrown to the Error Boundary. For 4xx errors, `throwOnError` returns `false`, so the error is handled locally by the component. The `retry: 2` option retries failed requests twice with exponential backoff. The Error Boundary catches the thrown 5xx error and displays its fallback.

### Real-World Cases

- **Search results:** Using TanStack Query with retry for transient failures and local error display for 4xx errors.
- **Form submissions:** Using RTK Query's `unwrap()` to handle mutation errors inline with `try`/`catch`.
- **Comments sections:** Using `useEffect` with `AbortController` to cancel stale requests and handle errors locally.
- **Dashboard widgets:** Using TanStack Query's `throwOnError` to send critical errors to a widget-level Error Boundary.
- **Infinite scroll:** Using TanStack Query's `useInfiniteQuery` with error handling for pagination failures.
- **Authentication flows:** Using RTK Query's `unwrap()` to handle login errors and display them inline.

---

## References

- React Official Documentation – Synchronizing with Effects: https://react.dev/learn/synchronizing-with-effects
- React Official Documentation – `useEffect` Reference: https://react.dev/reference/react/useEffect
- React Official Documentation – Error Boundaries: https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
- MDN Web Docs – `fetch()`: https://developer.mozilla.org/en-US/docs/Web/API/fetch
- MDN Web Docs – HTTP Status Codes: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status
- MDN Web Docs – `AbortController`: https://developer.mozilla.org/en-US/docs/Web/API/AbortController
- MDN Web Docs – `AbortSignal.timeout()`: https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/timeout_static
- Axios Documentation – Error Handling: https://axios-http.com/docs/handling_errors
- Axios Documentation – Interceptors: https://axios-http.com/docs/interceptors
- TanStack Query – Query Functions: https://tanstack.com/query/latest/docs/framework/react/guides/query-functions
- TanStack Query – Query Retries: https://tanstack.com/query/latest/docs/framework/react/guides/query-retries
- TanStack Query – Suspense: https://tanstack.com/query/latest/docs/framework/react/guides/suspense
- TanStack Query – `QueryErrorResetBoundary`: https://tanstack.com/query/latest/docs/framework/react/api-reference/QueryErrorResetBoundary
- RTK Query – Error Handling: https://redux-toolkit.js.org/rtk-query/usage/error-handling
- Redux Toolkit – `unwrap()`: https://redux-toolkit.js.org/rtk-query/usage/error-handling#handling-errors-with-a-custom-basequery
- Zod Documentation – Parsing: https://zod.dev/?id=parsing
- Zod Documentation – `safeParse`: https://zod.dev/?id=safeparse
- Yup Documentation: https://github.com/jquense/yup
- React Hook Form – Yup Resolver: https://github.com/react-hook-form/resolvers#yup
- Steve Kinney – Data Fetching and Runtime Validation: https://stevekinney.com/courses/react-typescript/data-fetching-and-runtime-validation
- Steve Kinney – Error Handling in React: https://stevekinney.com/courses/react-performance/error-handling
- Pratik Kamble – Unauthorized Access in React Apps: Best Practices for 401 and 403 Handling: https://www.linkedin.com/posts/pratikpkamble_react-javascript-frontend-activity-7437426119127232512-MCP9
- CoreUI – How to Protect API Requests in React: https://coreui.io/blog/how-to-protect-api-requests-in-react/
- web.dev – Fetch API: https://web.dev/articles/introduction-to-fetch
- web.dev – Interaction to Next Paint (INP): https://web.dev/articles/inp
- Epic React – Why React Error Boundaries Aren't Just Try/Catch for Components: https://www.epicreact.dev/why-react-error-boundaries-arent-just-try-catch-for-components