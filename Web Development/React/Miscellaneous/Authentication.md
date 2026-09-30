# Authentication Flows & State: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Authentication Flows & State in React refers to the architectural patterns, protocols, and state management strategies used to verify user identity, maintain authenticated sessions, and synchronise authentication state across a React application.

**Technical Definition:** Authentication in React SPAs involves integrating identity protocols (OAuth 2.0, OpenID Connect) with client-side state management to handle the complete lifecycle of user identity: initial authentication via an Identity Provider (IdP), token acquisition and storage, session persistence across page reloads, silent token refresh, and global state synchronisation across components, tabs, and server/client boundaries. React applications are public clients that cannot securely store client secrets, necessitating the Authorization Code flow with PKCE. Authentication state must be treated as a first-class concern, with careful attention to token storage (memory vs. HttpOnly cookies), refresh token rotation, and cross-tab synchronisation. The choice between token-based (JWT) and session-based authentication has significant implications for security, scalability, and architecture.

**Beginner-Friendly Explanation:** When you log into a website, that's authentication. But behind the scenes, a lot is happening: your app talks to a login service (like Google or your company's login system), gets a special pass (a token), and then uses that pass to prove who you are on every request. Managing authentication state means keeping track of whether you're logged in, who you are, and making sure your pass doesn't expire without you noticing. It's like having a membership card that you need to show at every door, and sometimes you need to renew it.

### Key Characteristics

- **Protocol-Driven:** Authentication flows follow standardised protocols (OAuth 2.0, OIDC) that define how tokens are requested, issued, and validated.
- **Public Client Constraints:** React SPAs are public clients that cannot keep secrets, requiring PKCE (Proof Key for Code Exchange) for security.
- **Stateful vs. Stateless:** Token-based (JWT) authentication is stateless (server doesn't store sessions), while cookie-based sessions are stateful (server stores session data).
- **Lifecycle-Aware:** Authentication state has a full lifecycle: initial login, token storage, session persistence, silent refresh, and logout.
- **Cross-Boundary Synchronisation:** Auth state must be consistent across React components, browser tabs, and the server/client boundary (especially in Next.js).
- **Security-Critical:** Token storage decisions directly affect vulnerability to XSS and CSRF attacks.

### Prerequisites

- Solid understanding of React Hooks (`useState`, `useEffect`, `useContext`) and component lifecycle.
- Familiarity with HTTP fundamentals (headers, status codes, cookies).
- Basic understanding of Promises and asynchronous JavaScript.
- Awareness of React Context and state management libraries (Zustand, Redux).
- For Next.js: understanding of server components, middleware, and the App Router.

### Related Programming Areas

- **Web Security:** XSS, CSRF, token storage, and PKCE.
- **State Management:** React Context, Zustand, Redux, and cross-tab synchronisation.
- **HTTP Protocol:** Cookies, headers, CORS, and SameSite attributes.
- **Identity & Access Management (IAM):** OAuth 2.0, OIDC, SAML, and federated identity.
- **Server-Side Rendering:** Session handling in Next.js, Remix, and other SSR frameworks.
- **API Security:** Bearer tokens, scopes, and role-based access control (RBAC).

### Core Concepts / Features

1. Identity Providers & Protocols
2. Credential Handling
3. Token vs. Cookie Authentication
4. Session Lifecycle Management
5. Global Authentication State

---

## Core Concept 1: Identity Providers & Protocols

### Definitions

**Core Definition:** Identity Providers & Protocols refers to the external services and standardised communication protocols used to authenticate users in a React application, including OAuth 2.0, OpenID Connect (OIDC), and federated login providers (Google, GitHub, Apple, etc.).

**Technical Definition:** OAuth 2.0 is an authorisation framework that enables applications to obtain limited access to user accounts on an HTTP service. OpenID Connect (OIDC) is an identity layer built on top of OAuth 2.0 that adds authentication capabilities, providing an ID Token (a signed JWT) containing user identity claims. In a React SPA, the correct flow is the **Authorization Code flow with PKCE** (Proof Key for Code Exchange), defined in RFC 7636. RFC 9700 (the OAuth 2.0 Security Best Current Practice) states that public clients MUST use PKCE and SHOULD NOT use the implicit grant. The flow works as follows: the app generates a random `code_verifier`, sends its SHA-256 hash as a `code_challenge` in the authorization request, and presents the original verifier when exchanging the authorization code for tokens. Libraries such as `oidc-client-ts` with `react-oidc-context` handle PKCE automatically. Federated login extends this by allowing users to authenticate via third-party IdPs (Google, GitHub, Apple) through the same OIDC flow.

**Beginner-Friendly Explanation:** Instead of building your own login system, you can let users log in with their Google, GitHub, or Apple account. OAuth2 and OIDC are the standards that make this possible. When a user clicks "Log in with Google," your app sends them to Google, Google verifies who they are, and then sends them back to your app with a special token that proves they're logged in. The key security trick (PKCE) ensures that even if someone intercepts the token, they can't use it without the secret code your app generated.

### Purposes

- To authenticate users via trusted third-party identity providers without managing passwords.
- To implement standardised, secure authentication flows using OAuth 2.0 and OIDC.
- To support federated login (Google, GitHub, Apple, Microsoft Entra ID, etc.) with a single integration.
- To obtain ID Tokens containing user identity claims (name, email, profile picture).
- To protect against authorization code interception attacks using PKCE.
- To enable Single Sign-On (SSO) across multiple applications within an organisation.

### Syntax Rules and Structure

**General Syntax with `react-oidc-context` and `oidc-client-ts`:**
```jsx
// authConfig.js
import { WebStorageStateStore } from 'oidc-client-ts';

export const oidcConfig = {
  authority: 'https://your-idp.com',
  client_id: 'your-client-id',
  redirect_uri: 'http://localhost:3000/callback',
  response_type: 'code',
  scope: 'openid profile email offline_access',
  post_logout_redirect_uri: 'http://localhost:3000',
  userStore: new WebStorageStateStore({ store: window.sessionStorage }),
  automaticSilentRenew: true,
  loadUserInfo: true,
};
```

```jsx
// index.jsx
import { AuthProvider } from 'react-oidc-context';
import { oidcConfig } from './authConfig';

root.render(
  <AuthProvider {...oidcConfig}>
    <App />
  </AuthProvider>
);
```

```jsx
// App.jsx
import { useAuth } from 'react-oidc-context';

function App() {
  const auth = useAuth();

  if (auth.isLoading) return <p>Loading...</p>;
  if (auth.error) return <p>Error: {auth.error.message}</p>;
  if (!auth.isAuthenticated) {
    return <button onClick={() => auth.signinRedirect()}>Log in</button>;
  }

  return (
    <div>
      <p>Hello, {auth.user?.profile.name}</p>
      <button onClick={() => auth.signoutRedirect()}>Log out</button>
    </div>
  );
}
```

**Component Breakdown:**
- `authority`: The URL of the OIDC identity provider.
- `client_id`: The public identifier for your React SPA.
- `redirect_uri`: The URL the IdP redirects to after authentication.
- `response_type: 'code'`: Uses the Authorization Code flow.
- `scope: 'openid profile email offline_access'`: Requests identity claims and a refresh token.
- `automaticSilentRenew`: Automatically refreshes tokens before expiry.
- `useAuth()`: A Hook from `react-oidc-context` that provides `isAuthenticated`, `user`, `signinRedirect`, `signoutRedirect`, and more.

**General Syntax for Manual OAuth 2.0 + PKCE:**
```javascript
// Generate PKCE code verifier and challenge
function generateCodeVerifier() {
  const array = new Uint8Array(32);
  crypto.getRandomValues(array);
  return base64UrlEncode(array);
}

async function generateCodeChallenge(verifier) {
  const encoder = new TextEncoder();
  const data = encoder.encode(verifier);
  const digest = await crypto.subtle.digest('SHA-256', data);
  return base64UrlEncode(new Uint8Array(digest));
}

// Step 1: Redirect to authorization endpoint
async function redirectToAuth() {
  const verifier = generateCodeVerifier();
  const challenge = await generateCodeChallenge(verifier);
  sessionStorage.setItem('pkce_verifier', verifier);

  const params = new URLSearchParams({
    response_type: 'code',
    client_id: 'your-client-id',
    redirect_uri: 'http://localhost:3000/callback',
    scope: 'openid profile email',
    code_challenge: challenge,
    code_challenge_method: 'S256',
    state: generateRandomState(),
  });

  window.location.href = `https://your-idp.com/authorize?${params}`;
}

// Step 2: Exchange code for tokens (in callback route)
async function handleCallback(code) {
  const verifier = sessionStorage.getItem('pkce_verifier');
  const response = await fetch('https://your-idp.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code,
      redirect_uri: 'http://localhost:3000/callback',
      client_id: 'your-client-id',
      code_verifier: verifier,
    }),
  });
  return response.json();
}
```

**Component Breakdown:**
- `generateCodeVerifier()`: Creates a random, high-entropy string.
- `generateCodeChallenge(verifier)`: Hashes the verifier using SHA-256 and base64url-encodes it.
- `code_challenge_method: 'S256'`: Specifies the SHA-256 hashing method.
- `sessionStorage.setItem('pkce_verifier', verifier)`: Stores the verifier for the token exchange.
- `code_verifier`: Sent during the token exchange to prove the same client initiated the flow.

**Syntax Rules:**
- Always use the Authorization Code flow with PKCE for React SPAs. The implicit flow is deprecated and SHOULD NOT be used.
- Never ship a client secret in a React bundle; the browser build is public.
- Store the PKCE `code_verifier` in `sessionStorage` (not `localStorage`) to limit exposure.
- Use the `offline_access` scope to request a refresh token for silent renewal.
- Validate the `state` parameter in the callback to prevent CSRF attacks.
- Use `automaticSilentRenew: true` for seamless token refresh.
- For enterprise SSO, use a broker (e.g., SSOJet, WorkOS) to support multiple IdPs without custom integrations.

**Constraints and Limitations:**
- PKCE requires `crypto.subtle` (available in modern browsers; not available in insecure contexts).
- Implicit flow is deprecated and should not be used for new applications.
- `react-oidc-context` relies on `oidc-client-ts`, which adds bundle size.
- Silent token renewal via iframe may be blocked by third-party cookie restrictions.
- Token storage in `localStorage` is vulnerable to XSS; consider in-memory storage or a Backend-For-Frontend (BFF) for sensitive applications.
- Federated login requires the IdP to be configured to accept your `redirect_uri`.

### Annotated Code Examples

**Example 1: Complete OIDC Integration with `react-oidc-context`**

```jsx
// authConfig.js
import { WebStorageStateStore } from 'oidc-client-ts';

export const oidcConfig = {
  authority: 'https://accounts.google.com',
  client_id: 'YOUR_GOOGLE_CLIENT_ID.apps.googleusercontent.com',
  redirect_uri: 'http://localhost:3000/callback',
  response_type: 'code',
  scope: 'openid profile email',
  post_logout_redirect_uri: 'http://localhost:3000',
  userStore: new WebStorageStateStore({ store: window.sessionStorage }),
  automaticSilentRenew: true,
  loadUserInfo: true,
};

// index.jsx
import React from 'react';
import { createRoot } from 'react-dom/client';
import { AuthProvider } from 'react-oidc-context';
import { oidcConfig } from './authConfig';
import App from './App';

createRoot(document.getElementById('root')).render(
  <AuthProvider {...oidcConfig}>
    <App />
  </AuthProvider>
);

// App.jsx
import React from 'react';
import { useAuth } from 'react-oidc-context';
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';

function Callback() {
  const auth = useAuth();
  // The AuthProvider automatically processes the callback
  // This component just handles the redirect
  if (auth.isAuthenticated) {
    return <Navigate to="/dashboard" replace />;
  }
  return <p>Processing login...</p>;
}

function Dashboard() {
  const auth = useAuth();
  return (
    <div>
      <h1>Dashboard</h1>
      <p>Welcome, {auth.user?.profile.name}</p>
      <p>Email: {auth.user?.profile.email}</p>
      <button onClick={() => auth.signoutRedirect()}>Log out</button>
    </div>
  );
}

function ProtectedRoute({ children }) {
  const auth = useAuth();

  if (auth.isLoading) return <p>Loading authentication...</p>;
  if (auth.error) return <p>Auth error: {auth.error.message}</p>;
  if (!auth.isAuthenticated) {
    return (
      <div>
        <p>You must be logged in to view this page.</p>
        <button onClick={() => auth.signinRedirect()}>Log in with Google</button>
      </div>
    );
  }

  return children;
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/callback" element={<Callback />} />
        <Route
          path="/dashboard"
          element={
            <ProtectedRoute>
              <Dashboard />
            </ProtectedRoute>
          }
        />
        <Route path="*" element={<Navigate to="/dashboard" replace />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

**Expected Output:** The app redirects unauthenticated users to a login page with a "Log in with Google" button. Clicking it redirects to Google's OAuth consent screen. After authentication, the user is redirected back to `/callback`, then automatically to `/dashboard`, where their name and email are displayed. Clicking "Log out" redirects to Google's logout page and then back to the app.

**Why This Output Occurs:** The `AuthProvider` from `react-oidc-context` handles the entire OIDC Authorization Code + PKCE flow automatically. When the user clicks "Log in," `signinRedirect()` generates the PKCE code verifier and challenge, redirects to Google, and stores the verifier. Google authenticates the user and redirects back with an authorization code. The `AuthProvider` exchanges the code for tokens using the stored verifier, validates the ID Token, and updates the `auth` context. The `ProtectedRoute` component checks `auth.isAuthenticated` and either renders the dashboard or shows the login prompt. `automaticSilentRenew: true` ensures tokens are refreshed in the background.

### Real-World Cases

- **Enterprise SaaS applications:** Integrating with Okta, Microsoft Entra ID, or Google Workspace for SSO, allowing enterprise customers to use their existing identity infrastructure.
- **Consumer applications:** Offering "Log in with Google/GitHub/Apple" to reduce friction and eliminate password management.
- **Multi-tenant platforms:** Using OIDC with tenant detection based on email domain to route users to the correct IdP.
- **Regulated industries:** Using a Backend-For-Frontend (BFF) with OIDC to keep tokens server-side, meeting compliance requirements (HIPAA, PCI DSS).
- **Developer tools:** Using GitHub OAuth to authenticate developers and access their repositories.

---

## Core Concept 2: Credential Handling

### Definitions

**Core Definition:** Credential Handling refers to the implementation of standard authentication UX flows—Login, Registration, and Multi-Factor Authentication (MFA)—including the forms, validation, error handling, and state management required for each.

**Technical Definition:** Credential handling encompasses three primary flows: (1) **Login**: collecting username/email and password, submitting to an authentication endpoint, and handling success (token storage, redirect) or failure (inline error display). (2) **Registration**: collecting user details, validating inputs (client-side with Zod/Yup and server-side), creating the account, and typically logging the user in automatically. (3) **Multi-Factor Authentication (MFA)**: after primary credential verification, prompting the user for a second factor (TOTP code from an authenticator app, SMS code, or passkey). MFA is typically a two-step process: initiate (send code or present QR) and verify (validate the code). Modern MFA implementations support method selection when users have multiple factors enrolled. The UX must handle loading states, error states, rate limiting, and accessibility.

**Beginner-Friendly Explanation:** Credential handling is the user-facing side of authentication. It's the login form, the registration page, and the "enter the 6-digit code from your authenticator app" screen. A good implementation makes these flows smooth: showing loading spinners, clear error messages ("Incorrect password" not "Error 401"), and handling special cases like MFA. The goal is to make authentication feel effortless while keeping it secure.

### Purposes

- To collect and validate user credentials securely.
- To provide clear, accessible login and registration experiences.
- To implement MFA as an additional security layer after primary authentication.
- To handle authentication errors gracefully with inline, user-friendly messages.
- To support multiple MFA methods (TOTP, SMS, passkeys) with method selection.
- To integrate with authentication libraries and providers for pre-built UI components.

### Syntax Rules and Structure

**General Syntax for Login Form with React Hook Form and Zod:**
```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const loginSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

function LoginForm({ onSubmit }) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    setError,
  } = useForm({ resolver: zodResolver(loginSchema) });

  const handleLogin = async (data) => {
    try {
      await onSubmit(data);
    } catch (error) {
      if (error.status === 401) {
        setError('root', { message: 'Invalid email or password' });
      } else {
        setError('root', { message: 'An unexpected error occurred' });
      }
    }
  };

  return (
    <form onSubmit={handleSubmit(handleLogin)}>
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" type="email" {...register('email')} />
        {errors.email && <p role="alert">{errors.email.message}</p>}
      </div>
      <div>
        <label htmlFor="password">Password</label>
        <input id="password" type="password" {...register('password')} />
        {errors.password && <p role="alert">{errors.password.message}</p>}
      </div>
      {errors.root && <p role="alert">{errors.root.message}</p>}
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Log in'}
      </button>
    </form>
  );
}
```

**Component Breakdown:**
- `zodResolver(loginSchema)`: Integrates Zod validation with React Hook Form.
- `register('email')`: Registers the input with React Hook Form.
- `errors.email`: Contains validation errors for the email field.
- `setError('root', ...)`: Sets a form-level error for server-side failures.
- `isSubmitting`: Disables the submit button during the request.

**General Syntax for MFA Verification:**
```jsx
function MfaVerification({ challengeId, onSuccess }) {
  const [code, setCode] = useState('');
  const [error, setError] = useState(null);
  const [isPending, setIsPending] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsPending(true);
    setError(null);

    try {
      const response = await fetch('/api/mfa/verify', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ challenge_id: challengeId, code }),
      });

      if (!response.ok) {
        const data = await response.json();
        throw new Error(data.message || 'Verification failed');
      }

      onSuccess();
    } catch (err) {
      setError(err.message);
    } finally {
      setIsPending(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <h2>Two-Factor Authentication</h2>
      <p>Enter the 6-digit code from your authenticator app.</p>
      <input
        type="text"
        inputMode="numeric"
        pattern="[0-9]{6}"
        maxLength={6}
        value={code}
        onChange={(e) => setCode(e.target.value.replace(/\D/g, ''))}
        placeholder="000000"
        autoFocus
      />
      {error && <p role="alert">{error}</p>}
      <button type="submit" disabled={isPending || code.length !== 6}>
        {isPending ? 'Verifying...' : 'Verify'}
      </button>
    </form>
  );
}
```

**Component Breakdown:**
- `challengeId`: The server-provided challenge identifier from the MFA initiation step.
- `code`: The 6-digit code entered by the user.
- `inputMode="numeric"`: Shows a numeric keyboard on mobile devices.
- `pattern="[0-9]{6}"`: HTML5 validation for exactly 6 digits.
- `e.target.value.replace(/\D/g, '')`: Strips non-digit characters.
- `disabled={isPending || code.length !== 6}`: Prevents submission until a valid code is entered.

**Syntax Rules:**
- Use React Hook Form with Zod or Yup for form validation.
- Display validation errors inline, next to the relevant field.
- Use `setError('root', ...)` for form-level (server-side) errors.
- Disable the submit button during submission to prevent duplicate requests.
- For MFA, use `inputMode="numeric"` and `pattern="[0-9]{6}"` for TOTP codes.
- Support method selection when users have multiple MFA methods enrolled.
- Use `aria-live` regions or `role="alert"` for error announcements.
- Consider using pre-built UI components from Clerk, Auth0, or Coinbase's CDP React for faster implementation.

**Constraints and Limitations:**
- Client-side validation is not a substitute for server-side validation; always validate on both.
- MFA codes have a short validity window (typically 30 seconds for TOTP).
- SMS-based MFA is vulnerable to SIM-swapping attacks; prefer TOTP or passkeys.
- Rate limiting must be implemented server-side to prevent brute-force attacks.
- Password managers may not autofill across multiple steps (email → password → MFA).
- Accessibility requires careful handling of focus management and error announcements.

### Annotated Code Examples

**Example 1: Complete Login Flow with Error Handling**

```jsx
import React, { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const loginSchema = z.object({
  email: z.string().email('Please enter a valid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

function LoginForm({ onLoginSuccess }) {
  const [serverError, setServerError] = useState(null);

  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm({ resolver: zodResolver(loginSchema) });

  async function onSubmit(data) {
    setServerError(null);

    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });

      if (response.status === 401) {
        setServerError('Invalid email or password. Please try again.');
        return;
      }

      if (response.status === 429) {
        setServerError('Too many attempts. Please wait before trying again.');
        return;
      }

      if (!response.ok) {
        throw new Error('Login failed');
      }

      const { token, user } = await response.json();
      onLoginSuccess(token, user);
    } catch (error) {
      setServerError('An unexpected error occurred. Please try again.');
    }
  }

  return (
    <div style={{ maxWidth: '400px', margin: '0 auto', padding: '20px' }}>
      <h1>Log In</h1>

      <form onSubmit={handleSubmit(onSubmit)} noValidate>
        <div style={{ marginBottom: '16px' }}>
          <label htmlFor="email" style={{ display: 'block', marginBottom: '4px' }}>
            Email
          </label>
          <input
            id="email"
            type="email"
            {...register('email')}
            style={{ width: '100%', padding: '8px' }}
            aria-invalid={errors.email ? 'true' : 'false'}
            aria-describedby={errors.email ? 'email-error' : undefined}
          />
          {errors.email && (
            <p id="email-error" role="alert" style={{ color: 'red', fontSize: '14px' }}>
              {errors.email.message}
            </p>
          )}
        </div>

        <div style={{ marginBottom: '16px' }}>
          <label htmlFor="password" style={{ display: 'block', marginBottom: '4px' }}>
            Password
          </label>
          <input
            id="password"
            type="password"
            {...register('password')}
            style={{ width: '100%', padding: '8px' }}
            aria-invalid={errors.password ? 'true' : 'false'}
            aria-describedby={errors.password ? 'password-error' : undefined}
          />
          {errors.password && (
            <p id="password-error" role="alert" style={{ color: 'red', fontSize: '14px' }}>
              {errors.password.message}
            </p>
          )}
        </div>

        {serverError && (
          <div role="alert" style={{ color: 'red', marginBottom: '16px' }}>
            {serverError}
          </div>
        )}

        <button
          type="submit"
          disabled={isSubmitting}
          style={{
            width: '100%',
            padding: '10px',
            backgroundColor: isSubmitting ? '#ccc' : '#007bff',
            color: 'white',
            border: 'none',
            borderRadius: '4px',
            cursor: isSubmitting ? 'not-allowed' : 'pointer',
          }}
        >
          {isSubmitting ? 'Logging in...' : 'Log In'}
        </button>
      </form>
    </div>
  );
}

export default LoginForm;
```

**Expected Output:** The login form displays two fields (email and password) with inline validation errors. If the user enters an invalid email or a short password, error messages appear below the fields. If the server returns a 401, "Invalid email or password. Please try again." appears at the bottom. If the server returns a 429, "Too many attempts. Please wait before trying again." appears. On success, the `onLoginSuccess` callback is called with the token and user data.

**Why This Output Occurs:** The `zodResolver` validates the form inputs and populates `errors` with field-specific messages. The `onSubmit` function handles the server response: 401 and 429 are handled with specific messages, while other errors trigger a generic message. The `aria-invalid` and `aria-describedby` attributes ensure the form is accessible to screen readers. The submit button is disabled during submission to prevent duplicate requests.

**Example 2: MFA Flow with Method Selection**

```jsx
import React, { useState, useEffect } from 'react';

function MfaFlow({ userId, onComplete }) {
  const [step, setStep] = useState('loading');
  const [methods, setMethods] = useState([]);
  const [selectedMethod, setSelectedMethod] = useState(null);
  const [challengeId, setChallengeId] = useState(null);
  const [code, setCode] = useState('');
  const [error, setError] = useState(null);
  const [isPending, setIsPending] = useState(false);

  // Step 1: Fetch available MFA methods
  useEffect(() => {
    async function fetchMethods() {
      try {
        const response = await fetch(`/api/mfa/methods?userId=${userId}`);
        const data = await response.json();
        setMethods(data.methods);

        if (data.methods.length === 1) {
          initiateVerification(data.methods[0].id);
        } else {
          setStep('select');
        }
      } catch (err) {
        setError('Failed to load MFA methods.');
      }
    }

    fetchMethods();
  }, [userId]);

  // Step 2: Initiate verification for the selected method
  async function initiateVerification(methodId) {
    setIsPending(true);
    setError(null);

    try {
      const response = await fetch('/api/mfa/initiate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ userId, methodId }),
      });

      if (!response.ok) throw new Error('Failed to initiate MFA');

      const data = await response.json();
      setChallengeId(data.challenge_id);
      setSelectedMethod(methodId);
      setStep('verify');
    } catch (err) {
      setError(err.message);
    } finally {
      setIsPending(false);
    }
  }

  // Step 3: Verify the code
  async function handleVerify(e) {
    e.preventDefault();
    setIsPending(true);
    setError(null);

    try {
      const response = await fetch('/api/mfa/verify', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ challenge_id: challengeId, code }),
      });

      if (!response.ok) {
        const data = await response.json();
        throw new Error(data.message || 'Invalid code');
      }

      onComplete();
    } catch (err) {
      setError(err.message);
      setCode('');
    } finally {
      setIsPending(false);
    }
  }

  if (step === 'loading') return <p>Loading MFA options...</p>;

  if (step === 'select') {
    return (
      <div>
        <h2>Choose a verification method</h2>
        {methods.map((method) => (
          <button
            key={method.id}
            onClick={() => initiateVerification(method.id)}
            disabled={isPending}
          >
            {method.type === 'totp' ? 'Authenticator App' : 'SMS'}
          </button>
        ))}
        {error && <p role="alert">{error}</p>}
      </div>
    );
  }

  return (
    <div>
      <h2>Enter Verification Code</h2>
      <p>
        {selectedMethod === 'totp'
          ? 'Enter the 6-digit code from your authenticator app.'
          : 'Enter the 6-digit code sent to your phone.'}
      </p>

      <form onSubmit={handleVerify}>
        <input
          type="text"
          inputMode="numeric"
          pattern="[0-9]{6}"
          maxLength={6}
          value={code}
          onChange={(e) => setCode(e.target.value.replace(/\D/g, ''))}
          placeholder="000000"
          autoFocus
          style={{ fontSize: '24px', letterSpacing: '8px', textAlign: 'center' }}
        />
        {error && <p role="alert" style={{ color: 'red' }}>{error}</p>}
        <button type="submit" disabled={isPending || code.length !== 6}>
          {isPending ? 'Verifying...' : 'Verify'}
        </button>
        <button type="button" onClick={() => setStep('select')}>
          Use a different method
        </button>
      </form>
    </div>
  );
}

export default MfaFlow;
```

**Expected Output:** The component first loads the available MFA methods. If there's only one method, it automatically initiates verification. If there are multiple methods, the user sees a selection screen with "Authenticator App" and "SMS" buttons. After selecting a method, the verification screen appears with a 6-digit input field. Entering the correct code calls `onComplete`. Entering an incorrect code shows an error message and clears the input.

**Why This Output Occurs:** The component follows a three-step flow: (1) fetch methods, (2) initiate verification (which may require method selection), (3) verify the code. The `useEffect` automatically initiates verification if only one method exists. The `inputMode="numeric"` and `pattern` attributes optimise the input for numeric codes. The "Use a different method" button allows users to switch methods, which is important when multiple factors are enrolled.

### Real-World Cases

- **Consumer applications:** Standard email/password login with "Remember me" and password reset flows.
- **Enterprise applications:** SSO login with MFA enforced by the organisation's IdP.
- **Financial applications:** Step-up authentication for sensitive actions (e.g., large transactions) requiring MFA even after login.
- **Healthcare applications:** MFA with TOTP for HIPAA compliance.
- **E-commerce:** Guest checkout with optional account creation, and MFA for account changes.
- **Developer platforms:** GitHub-style OAuth login with MFA enforcement for organisation members.

---

## Core Concept 3: Token vs. Cookie Authentication

### Definitions

**Core Definition:** Token vs. Cookie Authentication is the architectural decision between storing authentication credentials in JSON Web Tokens (JWTs) managed by client-side JavaScript versus HTTP-only, secure session cookies managed by the browser.

**Technical Definition:** **JWT authentication** stores a signed, self-contained token (typically in `localStorage` or memory) that the client sends in the `Authorization: Bearer <token>` header. The server validates the token's signature without storing session state, making it stateless and scalable. However, tokens stored in `localStorage` are accessible to JavaScript and therefore vulnerable to XSS attacks. **HTTP-only cookie authentication** stores a session identifier in an `HttpOnly`, `Secure`, `SameSite` cookie. The browser automatically includes the cookie in requests, and JavaScript cannot access it, mitigating XSS token theft. This approach typically requires server-side session storage (database, Redis), making it stateful but more secure. As of 2026, the consensus for SPAs is that **HttpOnly cookies with server-side sessions is the first-choice default**, even for SPAs, while JWTs make sense for explicit needs such as API-first architectures or when a Backend-For-Frontend (BFF) is not feasible.

**Beginner-Friendly Explanation:** When you log in, your app needs to remember who you are. There are two main ways to do this. With tokens, your app gets a special digital key that it stores in the browser and shows on every request. With cookies, the browser itself stores a session ID and automatically sends it with every request—your app's JavaScript never sees it. Cookies are safer because hackers can't steal them via JavaScript, but they require the server to remember your session. Tokens are easier to scale but require extra care to keep safe.

### Purposes

- To choose the appropriate credential storage mechanism based on security requirements.
- To mitigate XSS and CSRF attacks through HttpOnly and SameSite cookie attributes.
- To enable stateless authentication for API-first and microservices architectures.
- To support session revocation and immediate logout with server-side sessions.
- To balance scalability (stateless JWT) with security (HttpOnly cookies).
- To integrate with Backend-For-Frontend (BFF) patterns for regulated industries.

### Syntax Rules and Structure

**General Syntax for HttpOnly Cookie (Server-Side, Node.js/Express):**
```javascript
// Login endpoint
app.post('/api/auth/login', async (req, res) => {
  const { email, password } = req.body;
  const user = await verifyCredentials(email, password);

  if (!user) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }

  // Create a session
  const sessionId = crypto.randomUUID();
  await sessionStore.set(sessionId, {
    userId: user.id,
    createdAt: Date.now(),
    expiresAt: Date.now() + 7 * 24 * 60 * 60 * 1000, // 7 days
  });

  // Set HttpOnly cookie
  res.cookie('session_id', sessionId, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 7 * 24 * 60 * 60 * 1000,
    path: '/',
  });

  res.json({ user: { id: user.id, name: user.name, email: user.email } });
});

// Logout endpoint
app.post('/api/auth/logout', (req, res) => {
  const sessionId = req.cookies.session_id;
  if (sessionId) {
    sessionStore.delete(sessionId);
  }
  res.clearCookie('session_id');
  res.json({ message: 'Logged out' });
});

// Auth middleware
function requireAuth(req, res, next) {
  const sessionId = req.cookies.session_id;
  if (!sessionId) return res.status(401).json({ message: 'Not authenticated' });

  const session = sessionStore.get(sessionId);
  if (!session || session.expiresAt < Date.now()) {
    return res.status(401).json({ message: 'Session expired' });
  }

  req.userId = session.userId;
  next();
}
```

**Component Breakdown:**
- `httpOnly: true`: Prevents JavaScript access to the cookie.
- `secure: true` (production): Ensures the cookie is only sent over HTTPS.
- `sameSite: 'lax'`: Prevents CSRF attacks while allowing top-level navigation.
- `maxAge`: Cookie expiration time.
- `sessionStore`: Server-side session storage (Redis, database, or memory).
- `res.clearCookie('session_id')`: Removes the cookie on logout.

**General Syntax for JWT with In-Memory Storage (Client-Side):**
```javascript
// authStore.js (Zustand)
import { create } from 'zustand';

export const useAuthStore = create((set) => ({
  accessToken: null, // In-memory only — not persisted
  user: null,

  setAuth: (accessToken, user) => set({ accessToken, user }),
  clearAuth: () => set({ accessToken: null, user: null }),
}));

// apiClient.js
import axios from 'axios';
import { useAuthStore } from './authStore';

const apiClient = axios.create({ baseURL: '/api' });

apiClient.interceptors.request.use((config) => {
  const token = useAuthStore.getState().accessToken;
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Token refresh interceptor
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      try {
        const { data } = await axios.post('/api/auth/refresh', {}, {
          withCredentials: true, // Send HttpOnly refresh cookie
        });
        useAuthStore.getState().setAuth(data.accessToken, data.user);
        originalRequest.headers.Authorization = `Bearer ${data.accessToken}`;
        return apiClient(originalRequest);
      } catch (refreshError) {
        useAuthStore.getState().clearAuth();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    return Promise.reject(error);
  }
);
```

**Component Breakdown:**
- `accessToken: null`: The access token is stored in memory only (not `localStorage`).
- `apiClient.interceptors.request`: Attaches the token to every request.
- `apiClient.interceptors.response`: Handles 401 errors by attempting a token refresh.
- `withCredentials: true`: Sends the HttpOnly refresh cookie to the refresh endpoint.
- The refresh token is stored in an HttpOnly cookie, while the access token is in memory.

**Syntax Rules:**
- For maximum security, store the refresh token in an HttpOnly cookie and the access token in memory.
- Never store JWTs in `localStorage` if you can avoid it; XSS can steal them.
- Use `SameSite=Lax` or `SameSite=Strict` for cookies to prevent CSRF.
- Always use `Secure` cookies in production (HTTPS only).
- For JWTs, use short-lived access tokens (15 minutes) with refresh token rotation.
- For cookies, implement server-side session invalidation for immediate logout.
- Use a Backend-For-Frontend (BFF) pattern for regulated industries where no tokens should reach the browser.

**Constraints and Limitations:**
- `localStorage` is vulnerable to XSS; `HttpOnly` cookies are not accessible via JavaScript.
- Cookies require CSRF protection (SameSite, CSRF tokens).
- JWTs cannot be revoked before expiry without a blocklist (which negates statelessness).
- Server-side sessions require session storage infrastructure (Redis, database).
- `SameSite=None` requires `Secure` and may be blocked by third-party cookie restrictions.
- Cross-origin requests require CORS configuration and `withCredentials: true`.

### Annotated Code Examples

**Example 1: HttpOnly Cookie Authentication with Express**

```javascript
// server.js
import express from 'express';
import cookieParser from 'cookie-parser';
import crypto from 'crypto';

const app = express();
app.use(express.json());
app.use(cookieParser());

// In-memory session store (use Redis in production)
const sessions = new Map();

// Login
app.post('/api/auth/login', async (req, res) => {
  const { email, password } = req.body;

  // Simulate credential verification
  if (email !== 'user@example.com' || password !== 'password123') {
    return res.status(401).json({ message: 'Invalid credentials' });
  }

  const sessionId = crypto.randomUUID();
  sessions.set(sessionId, {
    userId: 'user-1',
    email,
    expiresAt: Date.now() + 7 * 24 * 60 * 60 * 1000,
  });

  res.cookie('session_id', sessionId, {
    httpOnly: true,
    secure: false, // Set to true in production (HTTPS)
    sameSite: 'lax',
    maxAge: 7 * 24 * 60 * 60 * 1000,
    path: '/',
  });

  res.json({ user: { id: 'user-1', email, name: 'Test User' } });
});

// Get current user
app.get('/api/auth/me', (req, res) => {
  const sessionId = req.cookies.session_id;
  if (!sessionId) return res.status(401).json({ message: 'Not authenticated' });

  const session = sessions.get(sessionId);
  if (!session || session.expiresAt < Date.now()) {
    return res.status(401).json({ message: 'Session expired' });
  }

  res.json({ user: { id: session.userId, email: session.email } });
});

// Logout
app.post('/api/auth/logout', (req, res) => {
  const sessionId = req.cookies.session_id;
  if (sessionId) sessions.delete(sessionId);
  res.clearCookie('session_id');
  res.json({ message: 'Logged out' });
});

app.listen(3001, () => console.log('Server running on port 3001'));
```

**Expected Output:** After logging in with the correct credentials, the server sets an `HttpOnly` cookie named `session_id`. Subsequent requests to `/api/auth/me` automatically include the cookie, and the server returns the user data. After logout, the cookie is cleared and the session is invalidated.

**Why This Output Occurs:** The `res.cookie()` call sets the cookie with `httpOnly: true`, making it inaccessible to JavaScript. The browser automatically sends the cookie with every subsequent request to the same origin. The `sessions` Map stores server-side session data, allowing the server to invalidate sessions on logout. The `sameSite: 'lax'` attribute prevents CSRF attacks from external sites.

**Example 2: JWT with In-Memory Access Token and HttpOnly Refresh Cookie**

```javascript
// server.js (refresh endpoint)
app.post('/api/auth/refresh', (req, res) => {
  const refreshToken = req.cookies.refresh_token;

  if (!refreshToken) {
    return res.status(401).json({ message: 'No refresh token' });
  }

  try {
    // Verify refresh token (JWT or opaque token)
    const payload = jwt.verify(refreshToken, REFRESH_SECRET);

    // Rotate refresh token (issue a new one)
    const newRefreshToken = jwt.sign(
      { userId: payload.userId, tokenVersion: payload.tokenVersion + 1 },
      REFRESH_SECRET,
      { expiresIn: '7d' }
    );

    const newAccessToken = jwt.sign(
      { userId: payload.userId },
      ACCESS_SECRET,
      { expiresIn: '15m' }
    );

    res.cookie('refresh_token', newRefreshToken, {
      httpOnly: true,
      secure: true,
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000,
      path: '/api/auth/refresh', // Restrict to refresh endpoint
    });

    res.json({ accessToken: newAccessToken });
  } catch (error) {
    res.clearCookie('refresh_token');
    res.status(401).json({ message: 'Invalid refresh token' });
  }
});
```

**Expected Output:** When the access token expires, the client calls `/api/auth/refresh` with the HttpOnly refresh cookie. The server validates the refresh token, issues a new access token and a rotated refresh token, and sets the new refresh cookie. The client stores the new access token in memory and retries the original request.

**Why This Output Occurs:** The refresh token is stored in an `HttpOnly` cookie with `path: '/api/auth/refresh'`, meaning it is only sent to the refresh endpoint—not to every API endpoint. This minimises exposure. The access token is returned in the JSON response and stored in memory by the client. Refresh token rotation ensures that if a refresh token is stolen, it becomes useless after the next refresh.

### Real-World Cases

- **Banking applications:** Using HttpOnly cookies with server-side sessions for maximum security, often with a BFF to keep all tokens server-side.
- **API-first SaaS:** Using JWTs for stateless authentication across microservices.
- **E-commerce:** Using HttpOnly session cookies for the web app and JWTs for mobile apps.
- **Healthcare:** Using a BFF with HttpOnly cookies to meet HIPAA requirements.
- **Startups:** Using managed auth providers (Clerk, Auth0) that handle token storage decisions automatically.

---

## Core Concept 4: Session Lifecycle Management

### Definitions

**Core Definition:** Session Lifecycle Management is the practice of managing the complete lifecycle of an authenticated session, including token expiration, silent token refreshing via rotation, and absolute/idle session logouts.

**Technical Definition:** Session lifecycle management encompasses several mechanisms: (1) **Token Expiration**: Access tokens are short-lived (typically 15 minutes) to limit the window of opportunity for stolen tokens. (2) **Silent Token Refresh**: Before the access token expires, the client uses a refresh token (stored in an HttpOnly cookie) to obtain a new access token without user interaction. (3) **Refresh Token Rotation**: Each refresh operation issues a new refresh token and invalidates the old one, making stolen refresh tokens detectable and useless after a single use. (4) **Absolute Session Logout**: The session is terminated after a fixed maximum duration (e.g., 24 hours), regardless of activity. (5) **Idle Session Logout**: The session is terminated after a period of inactivity (e.g., 30 minutes). Implementing silent refresh requires careful handling to avoid infinite loops, and the router must not be recreated on every token refresh—a common bug where `useMemo` invalidations are triggered by reference churn in auth state.

**Beginner-Friendly Explanation:** Tokens don't last forever—they expire. When a token expires, you don't want to kick the user out and make them log in again. Instead, your app can silently ask the server for a new token using a special "refresh token." This happens in the background, so the user never notices. Session lifecycle management is about making this process smooth: refreshing tokens before they expire, handling refresh failures gracefully, and logging users out after a certain period of inactivity for security.

### Purposes

- To maintain authenticated sessions without requiring frequent re-login.
- To refresh access tokens silently in the background before they expire.
- To implement refresh token rotation for enhanced security.
- To enforce absolute and idle session timeouts for security compliance.
- To handle token refresh failures gracefully (redirect to login).
- To prevent unnecessary re-renders and router recreation during token refresh.

### Syntax Rules and Structure

**General Syntax for Silent Token Refresh with Axios Interceptor:**
```javascript
import axios from 'axios';
import { useAuthStore } from './authStore';

let isRefreshing = false;
let failedQueue = [];

const processQueue = (error, token = null) => {
  failedQueue.forEach(({ resolve, reject }) => {
    if (error) reject(error);
    else resolve(token);
  });
  failedQueue = [];
};

const apiClient = axios.create({ baseURL: '/api' });

apiClient.interceptors.request.use((config) => {
  const token = useAuthStore.getState().accessToken;
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
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
        const { data } = await axios.post('/api/auth/refresh', {}, {
          withCredentials: true,
        });

        useAuthStore.getState().setAuth(data.accessToken, data.user);
        processQueue(null, data.accessToken);

        originalRequest.headers.Authorization = `Bearer ${data.accessToken}`;
        return apiClient(originalRequest);
      } catch (refreshError) {
        processQueue(refreshError);
        useAuthStore.getState().clearAuth();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      } finally {
        isRefreshing = false;
      }
    }

    return Promise.reject(error);
  }
);

export default apiClient;
```

**Component Breakdown:**
- `isRefreshing`: A flag to prevent multiple simultaneous refresh requests.
- `failedQueue`: Holds requests that arrived while the refresh was in progress.
- `processQueue`: Resolves or rejects all queued requests after the refresh completes.
- `originalRequest._retry`: Prevents infinite loops if the refreshed token also fails.
- `useAuthStore.getState().setAuth(...)`: Updates the in-memory access token.
- `withCredentials: true`: Sends the HttpOnly refresh cookie.

**General Syntax for Idle Session Timeout:**
```jsx
import { useEffect, useRef } from 'react';
import { useAuthStore } from './authStore';

const IDLE_TIMEOUT_MS = 30 * 60 * 1000; // 30 minutes

function useIdleTimeout() {
  const clearAuth = useAuthStore((s) => s.clearAuth);
  const timeoutRef = useRef(null);

  useEffect(() => {
    function resetTimer() {
      if (timeoutRef.current) clearTimeout(timeoutRef.current);
      timeoutRef.current = setTimeout(() => {
        clearAuth();
        window.location.href = '/login?reason=idle';
      }, IDLE_TIMEOUT_MS);
    }

    const events = ['mousedown', 'keydown', 'scroll', 'touchstart'];
    events.forEach((event) => window.addEventListener(event, resetTimer));
    resetTimer();

    return () => {
      if (timeoutRef.current) clearTimeout(timeoutRef.current);
      events.forEach((event) => window.removeEventListener(event, resetTimer));
    };
  }, [clearAuth]);
}
```

**Component Breakdown:**
- `IDLE_TIMEOUT_MS`: The duration of inactivity before logout.
- `resetTimer`: Resets the timeout on user activity.
- `events`: User activity events that reset the timer.
- `clearAuth()`: Clears the authentication state.
- Cleanup removes event listeners and clears the timer.

**Syntax Rules:**
- Use a queue to handle multiple simultaneous 401s during token refresh.
- Use the `_retry` flag to prevent infinite refresh loops.
- Store the access token in memory only (not `localStorage`).
- Use `withCredentials: true` to send the HttpOnly refresh cookie.
- Implement both absolute and idle session timeouts for security compliance.
- Preserve reference stability for auth state to prevent router recreation.
- Handle refresh failures by clearing auth state and redirecting to login.

**Constraints and Limitations:**
- Token refresh requires the refresh token to be valid and not expired.
- Refresh token rotation invalidates the old token; if the new token isn't stored, the user is logged out.
- Silent refresh via iframe may be blocked by third-party cookie restrictions.
- Idle timeout must account for multiple tabs; use `localStorage` events for cross-tab synchronisation.
- Router recreation during token refresh can cause navigation state loss.

### Annotated Code Examples

**Example 1: Complete Silent Refresh with Queue**

```javascript
// apiClient.js
import axios from 'axios';

const apiClient = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000,
});

let isRefreshing = false;
let failedQueue = [];

function processQueue(error, token = null) {
  failedQueue.forEach(({ resolve, reject }) => {
    if (error) reject(error);
    else resolve(token);
  });
  failedQueue = [];
}

apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
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
        const { data } = await axios.post(
          'https://api.example.com/auth/refresh',
          {},
          { withCredentials: true }
        );

        localStorage.setItem('accessToken', data.accessToken);
        processQueue(null, data.accessToken);

        originalRequest.headers.Authorization = `Bearer ${data.accessToken}`;
        return apiClient(originalRequest);
      } catch (refreshError) {
        processQueue(refreshError);
        localStorage.removeItem('accessToken');
        window.location.href = '/login';
        return Promise.reject(refreshError);
      } finally {
        isRefreshing = false;
      }
    }

    return Promise.reject(error);
  }
);

export default apiClient;
```

**Expected Output:** When multiple API requests fail with 401 simultaneously, only one refresh request is made. The other requests are queued and retried with the new token once the refresh completes. If the refresh fails, all queued requests are rejected, and the user is redirected to login.

**Why This Output Occurs:** The `isRefreshing` flag ensures only one refresh request at a time. The `failedQueue` holds requests that arrived during the refresh. The `processQueue` function resolves or rejects all queued requests after the refresh completes or fails. The `_retry` flag prevents infinite loops. This pattern is essential for applications that make multiple concurrent API calls.

**Example 2: Cross-Tab Auth Synchronisation with Zustand Persist**

```javascript
// authStore.js
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

export const useAuthStore = create(
  persist(
    (set, get) => ({
      user: null,
      accessToken: null,
      isAuthenticated: false,

      setAuth: (accessToken, user) => set({
        accessToken,
        user,
        isAuthenticated: true,
      }),

      clearAuth: () => set({
        accessToken: null,
        user: null,
        isAuthenticated: false,
      }),

      // Listen for storage events from other tabs
      syncFromStorage: () => {
        const state = get();
        // The persist middleware automatically syncs via localStorage
        // This method can be used for custom sync logic
      },
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({
        // Only persist non-sensitive data; keep accessToken in memory
        user: state.user,
        isAuthenticated: state.isAuthenticated,
      }),
      onRehydrateStorage: () => (state) => {
        // Optionally fetch a new access token on rehydration
        console.log('Auth state rehydrated from storage');
      },
    }
  )
);

// useCrossTabSync.js
import { useEffect } from 'react';
import { useAuthStore } from './authStore';

export function useCrossTabSync() {
  useEffect(() => {
    function handleStorageChange(event) {
      if (event.key === 'auth-storage') {
        // Zustand persist automatically updates the store
        // This handler is for additional custom logic (e.g., redirect)
        const newState = JSON.parse(event.newValue);
        if (!newState?.state?.isAuthenticated) {
          window.location.href = '/login';
        }
      }
    }

    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, []);
}
```

**Expected Output:** When a user logs out in one tab, the `auth-storage` key in `localStorage` is updated. The `storage` event fires in other tabs, and the `useCrossTabSync` Hook redirects them to the login page. When a user logs in, other tabs are also updated.

**Why This Output Occurs:** The Zustand `persist` middleware automatically syncs state to `localStorage`. The `storage` event is fired by the browser when `localStorage` is modified in another tab. The `useCrossTabSync` Hook listens for this event and handles the logout redirect. The `partialize` option ensures only non-sensitive data (user, isAuthenticated) is persisted, while the access token remains in memory.

### Real-World Cases

- **Banking applications:** Using short-lived access tokens (5 minutes) with silent refresh and absolute session timeouts (15 minutes) for security.
- **SaaS dashboards:** Using 15-minute access tokens with silent refresh and 30-minute idle timeouts.
- **E-commerce:** Using 1-hour sessions with silent refresh and "Remember me" for extended sessions.
- **Healthcare applications:** Using strict idle timeouts (10 minutes) for HIPAA compliance.
- **Multi-tab applications:** Syncing logout across all tabs using `storage` events.

---

## Core Concept 5: Global Authentication State

### Definitions

**Core Definition:** Global Authentication State refers to the architecture and patterns used to make authentication state (user, token, isAuthenticated) available throughout the React component tree and consistent across tabs, windows, and server/client boundaries.

**Technical Definition:** Global authentication state is typically managed through React Context (for simplicity and SSR compatibility) or Zustand (for flexibility and persistence middleware). The `AuthProvider` wraps the application and exposes an `auth` context via a `useAuth()` Hook. For Next.js applications, authentication state must be synchronised across the server/client boundary: server components can read cookies directly, while client components use the context. Next.js Middleware runs on the edge before a request is completed, enabling route protection based on cookies. Cross-tab synchronisation is achieved through the `storage` event (for `localStorage`) or the Broadcast Channel API. The key challenge is maintaining reference stability for auth state objects (especially roles arrays) to prevent unnecessary re-renders and router recreation during periodic token refresh.

**Beginner-Friendly Explanation:** When you log in, every part of your app needs to know you're logged in—the header, the sidebar, the protected routes. Global authentication state is how you share that information. You wrap your app in an "Auth Provider" that holds the current user and token, and any component can ask "Am I logged in?" using a simple Hook. For Next.js, you also need to make sure the server knows you're logged in, which is where cookies and middleware come in.

### Purposes

- To make authentication state available to every component in the application.
- To synchronise auth state across browser tabs and windows.
- To integrate with Next.js Middleware for edge-level route protection.
- To prevent unnecessary re-renders by maintaining reference stability.
- To provide a single source of truth for user identity and permissions.
- To handle SSR hydration correctly without mismatches.

### Syntax Rules and Structure

**General Syntax with React Context:**
```jsx
// AuthContext.jsx
import { createContext, useContext, useState, useCallback, useMemo } from 'react';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [accessToken, setAccessToken] = useState(null);

  const login = useCallback((token, userData) => {
    setAccessToken(token);
    setUser(userData);
  }, []);

  const logout = useCallback(() => {
    setAccessToken(null);
    setUser(null);
  }, []);

  // Memoise the context value to prevent unnecessary re-renders
  const value = useMemo(() => ({
    user,
    accessToken,
    isAuthenticated: !!accessToken,
    login,
    logout,
  }), [user, accessToken, login, logout]);

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

**Component Breakdown:**
- `createContext(null)`: Creates the context with a default value.
- `AuthProvider`: The provider component that holds the auth state.
- `useMemo`: Memoises the context value to prevent re-renders when auth state hasn't changed.
- `useCallback`: Memoises the `login` and `logout` functions.
- `useAuth()`: A custom Hook that throws if used outside the provider.

**General Syntax with Zustand:**
```javascript
// authStore.js
import { create } from 'zustand';

export const useAuthStore = create((set) => ({
  user: null,
  accessToken: null,
  isAuthenticated: false,

  setAuth: (accessToken, user) => set({
    accessToken,
    user,
    isAuthenticated: true,
  }),

  clearAuth: () => set({
    accessToken: null,
    user: null,
    isAuthenticated: false,
  }),

  // Preserve roles reference to prevent unnecessary re-renders
  updateRoles: (newRoles) => set((state) => {
    if (
      state.user?.roles?.length === newRoles.length &&
      state.user.roles.every((role, i) => role === newRoles[i])
    ) {
      return state; // No change — preserve reference
    }
    return { user: { ...state.user, roles: newRoles } };
  }),
}));
```

**Component Breakdown:**
- `create((set) => ({ ... }))`: Creates the Zustand store.
- `setAuth`: Sets the auth state.
- `clearAuth`: Clears the auth state.
- `updateRoles`: Preserves the roles array reference if the content hasn't changed.
- Zustand state can be accessed outside React components via `useAuthStore.getState()`.

**General Syntax for Next.js Middleware:**
```javascript
// middleware.js
import { NextResponse } from 'next/server';

export function middleware(request) {
  const sessionCookie = request.cookies.get('session_id');
  const { pathname } = request.nextUrl;

  // Protect dashboard routes
  if (pathname.startsWith('/dashboard')) {
    if (!sessionCookie) {
      const loginUrl = new URL('/login', request.url);
      loginUrl.searchParams.set('from', pathname);
      return NextResponse.redirect(loginUrl);
    }
  }

  // Redirect authenticated users away from login
  if (pathname === '/login' && sessionCookie) {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/login'],
};
```

**Component Breakdown:**
- `request.cookies.get('session_id')`: Reads the HttpOnly cookie in the middleware (server-side).
- `request.nextUrl`: The requested URL.
- `NextResponse.redirect()`: Redirects unauthenticated users to login.
- `config.matcher`: Specifies which routes the middleware applies to.
- Middleware runs on the edge before the request is completed.

**Syntax Rules:**
- Use React Context for simple, SSR-compatible auth state.
- Use Zustand for flexible auth state with persistence and cross-tab sync.
- Memoise context values to prevent unnecessary re-renders.
- Preserve reference stability for auth objects (especially roles) to prevent router recreation.
- In Next.js, use middleware for route protection at the edge.
- Server components read cookies directly; client components use the context.
- Use `storage` events or Broadcast Channel API for cross-tab synchronisation.
- Never store sensitive tokens in `localStorage` if using Zustand persist; keep them in memory.

**Constraints and Limitations:**
- React Context causes all consumers to re-render when the context value changes.
- Zustand persist may cause hydration mismatches in SSR if not configured correctly.
- Next.js Middleware runs on the edge and cannot access the full session store; it can only read cookies.
- Cross-tab sync via `storage` events only works for `localStorage`, not `sessionStorage`.
- Reference churn in auth state (e.g., new roles arrays on every refresh) can cause performance issues.
- The `useAuth` Hook must be used within the provider; otherwise, it throws.

### Annotated Code Examples

**Example 1: React Context with Reference Stability**

```jsx
// AuthContext.jsx
import React, { createContext, useContext, useState, useCallback, useMemo, useRef } from 'react';

const AuthContext = createContext(null);

// Helper to preserve roles reference when content is unchanged
function preserveRolesReference(prevRoles, newRoles) {
  if (
    prevRoles &&
    prevRoles.length === newRoles.length &&
    prevRoles.every((role, i) => role === newRoles[i])
  ) {
    return prevRoles; // Return the previous reference
  }
  return newRoles;
}

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [accessToken, setAccessToken] = useState(null);
  const userRef = useRef(user);

  const setAuth = useCallback((token, userData) => {
    setAccessToken(token);
    setUser((prevUser) => {
      // Preserve roles reference if content unchanged
      const roles = prevUser
        ? preserveRolesReference(prevUser.roles, userData.roles)
        : userData.roles;

      userRef.current = { ...userData, roles };
      return userRef.current;
    });
  }, []);

  const logout = useCallback(() => {
    setAccessToken(null);
    setUser(null);
  }, []);

  const value = useMemo(() => ({
    user,
    accessToken,
    isAuthenticated: !!accessToken,
    setAuth,
    logout,
  }), [user, accessToken, setAuth, logout]);

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
}
```

**Expected Output:** The `AuthProvider` exposes the auth state via context. When the auth state is updated (e.g., via token refresh), the `roles` array reference is preserved if the role content hasn't changed. This prevents downstream components (like routers that depend on `user.roles`) from unnecessarily re-rendering.

**Why This Output Occurs:** The `preserveRolesReference` function compares the previous and new roles arrays. If they are structurally equal, it returns the previous reference, preventing reference churn. The `useMemo` on the context value ensures the context only changes when the actual auth state changes. This is a production-grade pattern used to avoid router recreation during silent token refresh.

**Example 2: Next.js Middleware + Client Auth Sync**

```javascript
// middleware.js (runs on the edge)
import { NextResponse } from 'next/server';

export function middleware(request) {
  const token = request.cookies.get('access_token')?.value;

  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    if (!token) {
      const url = new URL('/login', request.url);
      url.searchParams.set('from', request.nextUrl.pathname);
      return NextResponse.redirect(url);
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*'],
};

// app/providers.jsx (client component)
'use client';
import { createContext, useContext, useState, useEffect } from 'react';

const AuthContext = createContext(null);

export function AuthProvider({ children, initialUser }) {
  const [user, setUser] = useState(initialUser);
  const [isLoading, setIsLoading] = useState(!initialUser);

  useEffect(() => {
    if (initialUser) return;

    // Fetch session from server
    async function fetchSession() {
      try {
        const res = await fetch('/api/auth/me');
        if (res.ok) {
          const data = await res.json();
          setUser(data.user);
        }
      } catch (err) {
        console.error('Session fetch failed:', err);
      } finally {
        setIsLoading(false);
      }
    }

    fetchSession();
  }, [initialUser]);

  return (
    <AuthContext.Provider value={{ user, isLoading, setUser }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}

// app/layout.jsx (server component)
import { cookies } from 'next/headers';
import { AuthProvider } from './providers';

export default async function RootLayout({ children }) {
  const cookieStore = await cookies();
  const token = cookieStore.get('access_token')?.value;

  let initialUser = null;
  if (token) {
    // Fetch user from server using the token
    try {
      const res = await fetch(`${process.env.API_URL}/auth/me`, {
        headers: { Authorization: `Bearer ${token}` },
        cache: 'no-store',
      });
      if (res.ok) {
        initialUser = (await res.json()).user;
      }
    } catch (err) {
      console.error('Failed to fetch initial user:', err);
    }
  }

  return (
    <html>
      <body>
        <AuthProvider initialUser={initialUser}>
          {children}
        </AuthProvider>
      </body>
    </html>
  );
}
```

**Expected Output:** The middleware protects `/dashboard` routes by checking for the `access_token` cookie. Unauthenticated users are redirected to `/login` with the intended destination preserved. The `AuthProvider` receives `initialUser` from the server component, so the initial render is already authenticated (no flash of unauthenticated content). On the client, if `initialUser` is not provided (e.g., after a client-side navigation), the provider fetches the session from `/api/auth/me`.

**Why This Output Occurs:** Next.js Middleware runs on the edge before the request reaches the page, providing fast route protection. The server component in `layout.jsx` reads the cookie and fetches the initial user, passing it to the client `AuthProvider`. This ensures SSR and client hydration are consistent, avoiding hydration mismatches. The client provider fetches the session only when `initialUser` is not provided.

### Real-World Cases

- **Next.js applications:** Using middleware for route protection and server components for initial auth state.
- **Multi-tenant SaaS:** Using context or Zustand to manage tenant-specific auth state and roles.
- **E-commerce:** Syncing cart and auth state across tabs using Zustand persist and `storage` events.
- **Enterprise applications:** Using React Context with reference stability to prevent router recreation during token refresh.
- **Progressive Web Apps (PWAs):** Using Zustand persist for offline-capable auth state.

---

## References

- Okta SSO for React SPA: OIDC Integration Guide – Security Boulevard: https://securityboulevard.com/2026/07/okta-sso-for-react-spa-oidc-integration-guide/
- Choose Authentication for React Apps – TanStack Start Docs: https://tanstack.com/start/latest/docs/framework/react/guide/authentication-overview
- Auth Role Preservation and Router Re-creation Fix – GitHub Wiki: https://github.com/bcgov/nr-waste-plus/wiki/Auth-Role-Preservation-and-Router-Re-creation-Fix-20260416
- Full Frontend Authentication System with AuthContext – GitHub Issue: https://github.com/StarShopCr/StarShop-Frontend/issues/153
- VerifyMfa Overview – Coinbase Developer Documentation: https://docs.cdp.coinbase.com/sdks/cdp-sdks-v2/frontend/@coinbase/cdp-react/Components/VerifyMfa.README
- Managed Authentication Services Comparison – GitHub: https://github.com/ancoleman/ai-design-components/blob/main/skills/securing-authentication/references/managed-auth-comparison.md
- OAuth 2.0 Security Best Current Practice (RFC 9700) – IETF: https://datatracker.ietf.org/doc/html/rfc9700
- Proof Key for Code Exchange (PKCE) – RFC 7636: https://datatracker.ietf.org/doc/html/rfc7636
- OpenID Connect Core 1.0 – OpenID Foundation: https://openid.net/specs/openid-connect-core-1_0.html
- Auth0 React SDK Documentation: https://auth0.com/docs/libraries/auth0-react
- Clerk React SDK Documentation: https://clerk.com/docs/quickstarts/react
- Firebase Auth for Web Documentation: https://firebase.google.com/docs/auth/web/start
- Next.js Middleware Documentation: https://nextjs.org/docs/app/building-your-application/routing/middleware
- React Router Authentication Guide: https://reactrouter.com/en/main/start/tutorial
- Zustand Persist Middleware Documentation: https://zustand.docs.pmnd.rs/integrations/persisting-store-data
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP JWT Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
- MDN Web Docs – HTTP Cookies: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
- MDN Web Docs – SameSite Cookies: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
- web.dev – SameSite Cookies Explained: https://web.dev/articles/samesite-cookies-explained
- Auth0 – Online Refresh Tokens: https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation
- Stack Overflow – Silent Token Refresh in React: https://stackoverflow.com/questions/73204172/silent-token-refresh-in-react