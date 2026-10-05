# Social Login & Authentication Middleware — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Social login is an authentication mechanism that allows users to sign in to an application using their existing credentials from a third-party identity provider (Google, GitHub, Microsoft, etc.), while authentication middleware is the Express-layer software that intercepts requests to verify identity and establish a user session.

**Technical Definition:** Social login is implemented through OAuth 2.0 and OpenID Connect (OIDC) protocols, where the application (Relying Party) delegates authentication to an external Identity Provider (IdP). Authentication middleware in Express is a function with the signature `(req, res, next)` that executes before route handlers, inspecting the request for credentials (session cookies, JWTs, OAuth tokens) and either populating `req.user` or rejecting the request with a 401/403 status.

**Beginner-Friendly Explanation:** Social login lets you click "Sign in with Google" instead of creating a new username and password. Authentication middleware is the behind-the-scenes security guard that checks your ID (session cookie or token) on every request to make sure you're allowed in. Passport.js is the most popular toolkit for handling social logins in Node.js, while modern SDKs from Auth0, Clerk, and Firebase provide managed alternatives.

### Key Characteristics

- **Delegated authentication:** The application trusts an external IdP to verify user identity.
- **Middleware pipeline:** Authentication happens in Express middleware, before route handlers.
- **Session or token-based:** Identity is maintained via server-side sessions (cookies) or stateless tokens (JWTs).
- **Strategy pattern:** Passport.js uses pluggable "strategies" for each provider (Google, GitHub, etc.).
- **Managed alternatives:** Auth0, Clerk, and Firebase abstract away the complexity of OAuth flows.

### Prerequisites

- **Node.js runtime** (v18 or higher).
- **Express.js installed:** `npm install express`.
- **Basic understanding of HTTP:** Cookies, headers, and the request–response cycle.
- **Familiarity with OAuth 2.0/OIDC concepts:** Authorization code flow, tokens, and scopes.
- **A registered OAuth application** with at least one provider (Google, GitHub, etc.).

### Related Programming Areas

- **OAuth 2.1 / OIDC:** The protocols underlying social login.
- **Session management:** `express-session` and cookie-based state.
- **JWT authentication:** Stateless token-based identity.
- **Authorization:** Role-based access control (RBAC) after authentication.
- **Security:** CSRF protection, session fixation prevention, and token validation.

### Core Concepts

1. **Passport.js Ecosystem** — Core architecture, middleware, serialization.
2. **Strategies** — Local, Google, GitHub, Azure AD.
3. **Modern Alternatives** — Auth0, Clerk, Firebase Auth SDKs.
4. **Session vs. Stateless** — Express-session vs. JWT validation.

---

## Core Concept 1: Passport.js Ecosystem

### Definitions

**Core Definition:** Passport.js is an authentication middleware for Node.js that provides a comprehensive set of strategies for authenticating requests using a username and password, social login, or other mechanisms.

**Technical Definition:** Passport is an authentication framework for Connect and Express that is extensible through "strategies." It does not mount routes or assume any particular database schema; instead, it exposes a single `passport.authenticate()` function that is used as route middleware. Passport maintains persistent login sessions by serializing the authenticated user to the session and deserializing the user on subsequent requests.

**Beginner-Friendly Explanation:** Passport is like a universal adapter for authentication. It doesn't care whether you're logging in with a password, Google, GitHub, or anything else — it provides a consistent interface (`passport.authenticate('strategy')`) and handles the messy details of session management for you.

### Purposes

- To provide a unified authentication middleware for Express applications.
- To abstract provider-specific authentication logic into reusable strategies.
- To manage login sessions through serialization and deserialization.
- To enable both session-based and stateless (JWT) authentication patterns.

### Sub-Feature 1.1: Core Architecture

#### Syntax Rules and Structure

```js
const passport = require('passport');
const session = require('express-session');

app.use(session({ secret: 'keyboard cat', resave: false, saveUninitialized: false }));
app.use(passport.initialize());   // Initialize Passport
app.use(passport.session());      // Enable session-based auth (optional)
```

| Middleware | Purpose |
|-----------|---------|
| `passport.initialize()` | Initializes Passport on every request. |
| `passport.session()` | Restores the login session from the session cookie. |

**Rules:**
- `passport.initialize()` must be called before any route that uses `passport.authenticate()`.
- `passport.session()` requires `express-session` (or compatible session middleware) to be mounted first.
- `passport.session()` is only needed for session-based authentication; omit it for stateless JWT.
- Passport must be configured with at least one strategy and serialization functions.

#### Annotated Code Example

```js
// passport-setup.js
const express = require('express');
const session = require('express-session');
const passport = require('passport');
const app = express();

// Step 1: Mount session middleware (required for passport.session())
app.use(session({
  secret: 'keyboard cat',
  resave: false,
  saveUninitialized: false
}));

// Step 2: Initialize Passport
app.use(passport.initialize());

// Step 3: Enable session restoration (only for session-based auth)
app.use(passport.session());

app.listen(3000, () => console.log('Passport initialized on 3000'));
```

**Expected Output:**
```
Passport initialized on 3000
```

**Why this output:** The three middleware are mounted in the correct order: session first (for cookie parsing), then `passport.initialize()` (to attach Passport to the request), then `passport.session()` (to restore the login session from the cookie). Without `express-session`, `passport.session()` would throw an error.

---

### Sub-Feature 1.2: Serialization and Deserialization

#### Definitions

**Core Definition:** Serialization determines what user information is stored in the session; deserialization retrieves the full user object from that stored information on subsequent requests.

**Technical Definition:** To maintain a login session, Passport serializes and deserializes user information to and from the session. The `serializeUser` function is called after login and determines what is stored in the session (typically just the user ID). The `deserializeUser` function is called with every request that has a session cookie, receiving the stored data and returning the full user object, which Passport assigns to `req.user`.

**Beginner-Friendly Explanation:** Think of serialization as writing a sticky note with just your user ID and sticking it in a cookie. Deserialization is reading that sticky note on your next visit and looking up your full profile from the database using that ID.

#### Syntax Rules and Structure

```js
passport.serializeUser((user, done) => {
  done(null, user.id);  // Store only the ID in the session
});

passport.deserializeUser(async (id, done) => {
  try {
    const user = await User.findById(id);
    done(null, user);   // Attach full user to req.user
  } catch (err) {
    done(err, null);
  }
});
```

| Function | When Called | Input | Output |
|----------|------------|-------|--------|
| `serializeUser` | After successful login | Full user object | Minimal data (e.g., ID) |
| `deserializeUser` | On every request with a cookie | Stored data (e.g., ID) | Full user object |

**Rules:**
- `serializeUser` is called just before the session is created.
- `deserializeUser` is called on every request that has a session cookie.
- The data stored should be minimal (ID only) to keep the session cookie small.
- `req.user` is set to the result of `deserializeUser`.

#### Annotated Code Example

```js
// serialize-deserialize.js
const passport = require('passport');

// Simulated database
const users = [{ id: 1, username: 'alice', email: 'alice@example.com' }];

// Store only the user ID in the session
passport.serializeUser((user, done) => {
  console.log('Serializing user:', user.id);
  done(null, user.id);
});

// Retrieve the full user from the database using the stored ID
passport.deserializeUser(async (id, done) => {
  console.log('Deserializing user ID:', id);
  try {
    const user = users.find(u => u.id === id);
    done(null, user);
  } catch (err) {
    done(err, null);
  }
});

// Simulate login and subsequent request
const mockUser = users[0];
passport.serializeUser(mockUser, (err, id) => {
  console.log('Stored in session:', id);
  passport.deserializeUser(id, (err, user) => {
    console.log('Restored user:', user);
  });
});
```

**Expected Output:**
```
Serializing user: 1
Stored in session: 1
Deserializing user ID: 1
Restored user: { id: 1, username: 'alice', email: 'alice@example.com' }
```

**Why this output:** `serializeUser` receives the full user object and stores only `user.id` (1) in the session. On the next request, `deserializeUser` receives the stored ID (1), looks up the full user in the database, and returns it. Passport assigns this to `req.user`, making the full profile available to route handlers.

#### Real-World Cases

- **Session-based web apps:** Store `userId` in the session and look up the user on each request.
- **Multi-database apps:** Store a composite key (e.g., `{ id, provider }`) for users from different sources.
- **Performance optimisation:** Use `deserializeUser` to cache user lookups in Redis or memory.

---

## Core Concept 2: Strategies

### Definitions

**Core Definition:** A Passport strategy is a pluggable authentication mechanism that implements a specific protocol (local password, OAuth 2.0, OpenID Connect) for verifying user identity.

**Technical Definition:** Strategies are packaged modules that implement the `passport-strategy` interface. Each strategy is configured with provider-specific options (client ID, secret, callback URL) and a `verify` callback that receives the credentials (access token, profile, etc.) and calls `done()` with the authenticated user. Strategies are registered with `passport.use(new Strategy(...))` and invoked with `passport.authenticate('strategy-name')`.

**Beginner-Friendly Explanation:** A strategy is like a different type of key for a lock. The local strategy uses a username and password. The Google strategy uses a Google account. The GitHub strategy uses a GitHub account. Passport doesn't care which key you use — it just needs a strategy to turn the key and tell it whether the user is authenticated.

### Purposes

- To authenticate users via different identity sources with a consistent interface.
- To abstract provider-specific OAuth flows into reusable modules.
- To enable multi-provider authentication in a single application.

### Sub-Feature 2.1: Local Strategy

#### Definitions

**Core Definition:** The Local strategy authenticates users using a username and password stored in the application's own database.

**Technical Definition:** The `passport-local` strategy authenticates users using a username and password. It requires a `verify` callback that accepts the credentials and calls `done()` with the user object if valid.

#### Syntax Rules and Structure

```js
const LocalStrategy = require('passport-local').Strategy;

passport.use(new LocalStrategy(
  async (username, password, done) => {
    const user = await User.findOne({ username });
    if (!user) return done(null, false, { message: 'Incorrect username' });
    if (!await bcrypt.compare(password, user.passwordHash)) {
      return done(null, false, { message: 'Incorrect password' });
    }
    return done(null, user);
  }
));
```

#### Annotated Code Example

```js
// local-strategy.js
const express = require('express');
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const bcrypt = require('bcrypt');
const session = require('express-session');
const app = express();

app.use(express.urlencoded({ extended: true }));
app.use(session({ secret: 'keyboard cat', resave: false, saveUninitialized: false }));
app.use(passport.initialize());
app.use(passport.session());

// Simulated user database
const users = [{
  id: 1,
  username: 'alice',
  passwordHash: '$2b$10$...' // bcrypt hash of "password123"
}];

passport.use(new LocalStrategy(
  async (username, password, done) => {
    const user = users.find(u => u.username === username);
    if (!user) return done(null, false, { message: 'User not found' });
    const valid = await bcrypt.compare(password, user.passwordHash);
    if (!valid) return done(null, false, { message: 'Wrong password' });
    return done(null, user);
  }
));

passport.serializeUser((user, done) => done(null, user.id));
passport.deserializeUser((id, done) => {
  done(null, users.find(u => u.id === id));
});

app.post('/login',
  passport.authenticate('local', { failureRedirect: '/login-failed' }),
  (req, res) => res.json({ message: 'Logged in', user: req.user })
);

app.listen(3000, () => console.log('Local strategy on 3000'));
```

**Expected Output (for `POST /login` with valid credentials):**
```json
{"message":"Logged in","user":{"id":1,"username":"alice"}}
```

**Expected Output (for invalid credentials):**
```
302 Redirect to /login-failed
```

**Why this output:** The Local strategy's verify callback looks up the user by username, compares the provided password against the stored hash using bcrypt, and calls `done(null, user)` on success. Passport then serializes the user ID into the session and redirects or responds. On failure, Passport redirects to the `failureRedirect` path.

---

### Sub-Feature 2.2: Google OAuth 2.0 Strategy

#### Definitions

**Core Definition:** The `passport-google-oauth20` strategy authenticates users using their Google account via OAuth 2.0.

**Technical Definition:** The Google authentication strategy authenticates users using a Google account and OAuth 2.0 tokens. The strategy requires a `verify` callback that receives the access token, optional refresh token, and profile (containing the authenticated user's Google profile). The callback must call `done()` providing a user to complete authentication. Two routes are needed: one to redirect to Google (`/auth/google`) and one to handle the callback (`/auth/google/callback`).

#### Syntax Rules and Structure

```js
const GoogleStrategy = require('passport-google-oauth20').Strategy;

passport.use(new GoogleStrategy({
  clientID: GOOGLE_CLIENT_ID,
  clientSecret: GOOGLE_CLIENT_SECRET,
  callbackURL: 'http://www.example.com/auth/google/callback'
}, (accessToken, refreshToken, profile, done) => {
  User.findOrCreate({ googleId: profile.id }, (err, user) => done(err, user));
}));

app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] }));
app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => res.redirect('/'));
```

| Route | Purpose |
|-------|---------|
| `GET /auth/google` | Redirects user to Google's consent screen. |
| `GET /auth/google/callback` | Handles the callback from Google after consent. |

#### Annotated Code Example

```js
// google-strategy.js
const express = require('express');
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;
const session = require('express-session');
const app = express();

app.use(session({ secret: 'keyboard cat', resave: false, saveUninitialized: false }));
app.use(passport.initialize());
app.use(passport.session());

// Simulated user store
const users = [];

passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: 'http://localhost:3000/auth/google/callback'
}, (accessToken, refreshToken, profile, done) => {
  // Find or create user in your database
  let user = users.find(u => u.googleId === profile.id);
  if (!user) {
    user = {
      id: users.length + 1,
      googleId: profile.id,
      name: profile.displayName,
      email: profile.emails?.[0]?.value
    };
    users.push(user);
  }
  return done(null, user);
}));

passport.serializeUser((user, done) => done(null, user.id));
passport.deserializeUser((id, done) => {
  done(null, users.find(u => u.id === id));
});

// Step 1: Redirect to Google
app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] }));

// Step 2: Handle callback
app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    res.json({ message: 'Google login successful', user: req.user });
  });

app.listen(3000, () => console.log('Google strategy on 3000'));
```

**Expected Output (after successful Google authentication):**
```json
{
  "message": "Google login successful",
  "user": {
    "id": 1,
    "googleId": "1234567890",
    "name": "Alice Johnson",
    "email": "alice@gmail.com"
  }
}
```

**Why this output:** The first route redirects the user to Google's OAuth consent screen. After the user approves, Google redirects back to `/auth/google/callback` with an authorization code, which Passport exchanges for tokens and a profile. The verify callback finds or creates the user in the local database and passes it to `done()`. Passport then serializes the user and completes the login.

---

### Sub-Feature 2.3: GitHub Strategy

#### Definitions

**Core Definition:** The `passport-github2` strategy authenticates users using their GitHub account via OAuth 2.0.

**Technical Definition:** The GitHub authentication strategy authenticates users using a GitHub account and OAuth 2.0 tokens. It requires a verify callback that accepts the access token, refresh token, and profile, and calls `done` providing a user. The strategy requires a `clientID`, `clientSecret`, and `callbackURL`.

#### Syntax Rules and Structure

```js
const GitHubStrategy = require('passport-github2').Strategy;

passport.use(new GitHubStrategy({
  clientID: GITHUB_CLIENT_ID,
  clientSecret: GITHUB_CLIENT_SECRET,
  callbackURL: 'http://127.0.0.1:3000/auth/github/callback'
}, (accessToken, refreshToken, profile, done) => {
  User.findOrCreate({ githubId: profile.id }, (err, user) => done(err, user));
}));

app.get('/auth/github',
  passport.authenticate('github', { scope: ['user:email'] }));
app.get('/auth/github/callback',
  passport.authenticate('github', { failureRedirect: '/login' }),
  (req, res) => res.redirect('/'));
```

#### Annotated Code Example

```js
// github-strategy.js
const express = require('express');
const passport = require('passport');
const GitHubStrategy = require('passport-github2').Strategy;
const session = require('express-session');
const app = express();

app.use(session({ secret: 'keyboard cat', resave: false, saveUninitialized: false }));
app.use(passport.initialize());
app.use(passport.session());

const users = [];

passport.use(new GitHubStrategy({
  clientID: process.env.GITHUB_CLIENT_ID,
  clientSecret: process.env.GITHUB_CLIENT_SECRET,
  callbackURL: 'http://localhost:3000/auth/github/callback'
}, (accessToken, refreshToken, profile, done) => {
  let user = users.find(u => u.githubId === profile.id);
  if (!user) {
    user = {
      id: users.length + 1,
      githubId: profile.id,
      username: profile.username,
      displayName: profile.displayName
    };
    users.push(user);
  }
  return done(null, user);
}));

passport.serializeUser((user, done) => done(null, user.id));
passport.deserializeUser((id, done) => done(null, users.find(u => u.id === id)));

app.get('/auth/github',
  passport.authenticate('github', { scope: ['user:email'] }));

app.get('/auth/github/callback',
  passport.authenticate('github', { failureRedirect: '/login' }),
  (req, res) => res.json({ message: 'GitHub login successful', user: req.user }));

app.listen(3000, () => console.log('GitHub strategy on 3000'));
```

**Expected Output:**
```json
{
  "message": "GitHub login successful",
  "user": { "id": 1, "githubId": "12345", "username": "alice", "displayName": "Alice" }
}
```

**Why this output:** The GitHub strategy follows the same pattern as Google: redirect to GitHub, handle the callback, find or create the user, and complete the login. The `scope: ['user:email']` option requests access to the user's email address.

---

### Sub-Feature 2.4: Azure AD Strategy

#### Definitions

**Core Definition:** The `passport-azure-ad` module provides OpenID Connect and Bearer strategies for authenticating with Microsoft Entra ID (formerly Azure Active Directory).

**Technical Definition:** `passport-azure-ad` is a collection of Passport strategies to help integrate with Azure Active Directory. It includes two strategies: `OIDCStrategy` (for web apps using OpenID Connect) and `BearerStrategy` (for APIs validating JWT access tokens). The OIDC strategy automatically handles signing key rollover by fetching and caching keys from the OpenID Connect discovery document.

#### Syntax Rules and Structure

```js
const OIDCStrategy = require('passport-azure-ad').OIDCStrategy;

passport.use(new OIDCStrategy({
  identityMetadata: `https://login.microsoftonline.com/${tenantId}/v2.0/.well-known/openid-configuration`,
  clientID: AZURE_CLIENT_ID,
  clientSecret: AZURE_CLIENT_SECRET,
  responseType: 'code id_token',
  redirectUrl: 'http://localhost:3000/auth/azure/callback',
  allowHttpForRedirectUrl: true,
  scope: ['openid', 'profile', 'email']
}, (iss, sub, profile, accessToken, refreshToken, done) => {
  done(null, profile);
}));
```

#### Annotated Code Example

```js
// azure-ad-strategy.js
const express = require('express');
const passport = require('passport');
const OIDCStrategy = require('passport-azure-ad').OIDCStrategy;
const session = require('express-session');
const app = express();

app.use(session({ secret: 'keyboard cat', resave: false, saveUninitialized: false }));
app.use(passport.initialize());
app.use(passport.session());

passport.use(new OIDCStrategy({
  identityMetadata: 'https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration',
  clientID: process.env.AZURE_CLIENT_ID,
  clientSecret: process.env.AZURE_CLIENT_SECRET,
  responseType: 'code id_token',
  redirectUrl: 'http://localhost:3000/auth/azure/callback',
  allowHttpForRedirectUrl: true,
  scope: ['openid', 'profile', 'email']
}, (iss, sub, profile, accessToken, refreshToken, done) => {
  return done(null, {
    id: sub,
    displayName: profile.displayName,
    email: profile._json.email
  });
}));

passport.serializeUser((user, done) => done(null, user));
passport.deserializeUser((user, done) => done(null, user));

app.get('/auth/azure',
  passport.authenticate('azuread-openidconnect', { failureRedirect: '/login' }));

app.get('/auth/azure/callback',
  passport.authenticate('azuread-openidconnect', { failureRedirect: '/login' }),
  (req, res) => res.json({ message: 'Azure AD login successful', user: req.user }));

app.listen(3000, () => console.log('Azure AD strategy on 3000'));
```

**Expected Output:**
```json
{
  "message": "Azure AD login successful",
  "user": {
    "id": "abc123",
    "displayName": "Alice Johnson",
    "email": "alice@contoso.com"
  }
}
```

**Why this output:** The OIDCStrategy redirects the user to Microsoft's login page. After authentication, Microsoft redirects back with an ID token. The strategy verifies the token's signature using keys fetched from the OpenID Connect discovery document, then calls the verify callback with the user's profile. The app stores the profile and completes the login.

---

## Core Concept 3: Modern Alternatives — Managed Authentication SDKs

### Definitions

**Core Definition:** Managed authentication SDKs (Auth0, Clerk, Firebase Auth) are commercial or cloud-hosted platforms that provide authentication as a service, abstracting away the complexity of OAuth flows, token management, and session handling.

**Technical Definition:** These SDKs provide Express middleware that handles the full OAuth/OIDC flow, including redirects, token exchange, session creation, and token verification. They typically expose a middleware function (e.g., `auth()`, `clerkMiddleware()`, `firebaseAuthMiddleware()`) and route-level guards (e.g., `requiresAuth()`, `requireAuth()`) that protect endpoints. They handle token validation, key rotation, and session management in the background.

**Beginner-Friendly Explanation:** Instead of wiring up Passport strategies yourself, you use a managed service that does the hard work. Auth0, Clerk, and Firebase are like hiring a professional security company instead of installing your own locks. They handle the login pages, token validation, and session management, and you just call a middleware function to protect your routes.

### Purposes

- To reduce the complexity and maintenance burden of OAuth/OIDC integration.
- To provide production-ready security without deep protocol expertise.
- To offer managed login UIs, user management, and MFA out of the box.
- To handle token validation, key rotation, and session management automatically.

### Sub-Feature 3.1: Auth0 Express SDK

#### Syntax Rules and Structure

```js
const { auth, requiresAuth } = require('express-openid-connect');

app.use(auth({
  issuerBaseURL: 'https://YOUR_DOMAIN.auth0.com',
  baseURL: 'http://localhost:3000',
  clientID: 'YOUR_CLIENT_ID',
  secret: 'YOUR_CLIENT_SECRET'
}));

// Protect a route
app.get('/profile', requiresAuth(), (req, res) => {
  res.json(req.oidc.user);
});
```

| Component | Breakdown |
|-----------|-----------|
| `auth()` | Middleware that handles session management and creates `/login`, `/logout`, `/callback` routes. |
| `requiresAuth()` | Middleware that requires authentication for specific routes. |
| `req.oidc.user` | Contains the authenticated user's profile. |

#### Annotated Code Example

```js
// auth0-express.js
const express = require('express');
const { auth, requiresAuth } = require('express-openid-connect');
const app = express();

app.use(auth({
  issuerBaseURL: process.env.AUTH0_ISSUER_BASE_URL,
  baseURL: process.env.AUTH0_BASE_URL,
  clientID: process.env.AUTH0_CLIENT_ID,
  secret: process.env.AUTH0_SECRET,
  authRequired: false,        // Protect routes individually
  auth0Logout: true
}));

// Public route
app.get('/', (req, res) => {
  res.json({ loggedIn: req.oidc.isAuthenticated() });
});

// Protected route
app.get('/profile', requiresAuth(), (req, res) => {
  res.json({
    message: 'Authenticated',
    user: req.oidc.user
  });
});

app.listen(3000, () => console.log('Auth0 Express SDK on 3000'));
```

**Expected Output (for `GET /profile` with valid session):**
```json
{
  "message": "Authenticated",
  "user": {
    "sub": "auth0|12345",
    "name": "Alice Johnson",
    "email": "alice@example.com",
    "picture": "https://example.com/alice.jpg"
  }
}
```

**Why this output:** The `auth()` middleware automatically creates `/login`, `/logout`, and `/callback` routes. When the user visits `/login`, they are redirected to Auth0's hosted login page. After authentication, the user is redirected back, a session is created, and `req.oidc.user` contains the decoded ID token claims. The `requiresAuth()` middleware ensures that only authenticated users can access `/profile`.

---

### Sub-Feature 3.2: Clerk Express SDK

#### Syntax Rules and Structure

```js
import { clerkMiddleware, requireAuth, getAuth } from '@clerk/express';

app.use(clerkMiddleware());  // Attach auth to all requests

// Protect a route
app.get('/protected', requireAuth(), (req, res) => {
  const { userId } = getAuth(req);
  res.json({ userId });
});
```

| Middleware | Purpose |
|-----------|---------|
| `clerkMiddleware()` | Checks cookies/headers for a session JWT and attaches auth to `req.auth`. |
| `requireAuth()` | Protects routes by redirecting unauthenticated users to sign-in. |
| `getAuth(req)` | Retrieves the authentication state from the request. |

**Rules:**
- `clerkMiddleware()` must be mounted globally before any route that uses `requireAuth()`.
- `requireAuth()` redirects unauthenticated users to the sign-in page (in the Express SDK; the Node SDK emitted an error).
- The `req.auth` object contains the user ID and session claims.

#### Annotated Code Example

```js
// clerk-express.js
import express from 'express';
import { clerkMiddleware, requireAuth, getAuth } from '@clerk/express';

const app = express();

// Attach Clerk auth to all requests
app.use(clerkMiddleware());

// Public route
app.get('/api/public', (req, res) => {
  res.json({ message: 'Public endpoint' });
});

// Protected route
app.get('/api/profile', requireAuth(), (req, res) => {
  const { userId } = getAuth(req);
  res.json({
    message: 'Authenticated',
    userId
  });
});

app.listen(3000, () => console.log('Clerk Express on 3000'));
```

**Expected Output (for `GET /api/profile` with valid session):**
```json
{
  "message": "Authenticated",
  "userId": "user_2abc123"
}
```

**Expected Output (for unauthenticated request):**
```
302 Redirect to Clerk sign-in page
```

**Why this output:** `clerkMiddleware()` checks the request's cookies and headers for a session JWT and attaches the auth object to `req.auth`. The `requireAuth()` middleware then checks if the user is authenticated; if not, it redirects to the Clerk sign-in page. If authenticated, `getAuth(req)` retrieves the user ID from the session.

---

### Sub-Feature 3.3: Firebase Auth Middleware

#### Syntax Rules and Structure

```js
const admin = require('firebase-admin');
const { firebaseAuthMiddleware, requireAuth } = require('@my-f-startup/firebase-auth-express');

admin.initializeApp();

app.use(firebaseAuthMiddleware());  // Validates tokens on every request

app.get('/protected', requireAuth((req, res) => {
  res.json({ uid: req.auth.uid });
}));
```

| Middleware | Purpose |
|-----------|---------|
| `firebaseAuthMiddleware()` | Extracts Bearer token, calls `verifyIdToken()`, sets `req.auth`. |
| `requireAuth(handler)` | Guards routes; returns 401 if `req.auth` is missing. |

**Rules:**
- The middleware extracts the `Authorization: Bearer <token>` header.
- It calls `admin.auth().verifyIdToken(token)` to validate the ID token.
- `req.auth` is set to `{ uid, token }` on success.
- Returns 401 on missing or invalid tokens.

#### Annotated Code Example

```js
// firebase-auth.js
import express from 'express';
import admin from 'firebase-admin';
import {
  firebaseAuthMiddleware,
  requireAuth,
  requireRole
} from '@my-f-startup/firebase-auth-express';

admin.initializeApp({ projectId: 'demo-project' });

const app = express();
app.use(express.json());

// Global authentication middleware
app.use(firebaseAuthMiddleware());

// Public route (no auth required)
app.get('/api/public', (req, res) => {
  res.json({ message: 'Public endpoint' });
});

// Protected route
app.get('/api/me', requireAuth((req, res) => {
  res.json({ uid: req.auth.uid });
}));

// Role-protected route
app.get('/api/admin', requireRole('admin', (req, res) => {
  res.json({ message: 'Admin access granted', uid: req.auth.uid });
}));

app.listen(3000, () => console.log('Firebase Auth on 3000'));
```

**Expected Output (for `GET /api/me` with valid Firebase ID token):**
```json
{"uid":"user_abc123"}
```

**Expected Output (for `GET /api/admin` without admin role):**
```
403 Forbidden
```

**Why this output:** The `firebaseAuthMiddleware()` runs on every request, extracting the Bearer token and calling `admin.auth().verifyIdToken()`. On success, `req.auth` is populated with the user's UID and token claims. The `requireAuth()` guard ensures the request is authenticated before executing the handler. The `requireRole('admin')` guard checks custom claims to enforce role-based access control.

---

## Core Concept 4: Session vs. Stateless

### Definitions

**Core Definition:** Session-based authentication stores user state on the server and uses a cookie to identify the session; stateless authentication embeds identity in a signed token (JWT) that is sent with every request.

**Technical Definition:** In session-based authentication, `express-session` creates a server-side session store (memory, Redis, MongoDB) and sets a session ID cookie in the browser. Passport serializes the user ID into this session. On subsequent requests, the cookie is sent, the session is looked up, and `deserializeUser` restores the user. In stateless authentication, a JWT is signed by the server and sent to the client, which stores it (usually in memory or an HTTP-only cookie). The server verifies the JWT signature on each request without any server-side session lookup.

**Beginner-Friendly Explanation:** Session-based auth is like a coat check at a restaurant — you get a ticket (cookie), and the restaurant holds your coat (session data) in the back. Stateless auth is like a signed ID card — you carry it with you, and anyone can verify it's authentic without calling the issuing office. Sessions are simpler and allow immediate logout; JWTs are better for distributed systems and mobile apps.

### Purposes

- To choose the right authentication architecture for the application's scale and requirements.
- To enable horizontal scaling with stateless tokens.
- To provide immediate logout and session revocation with server-side sessions.
- To support cross-domain and mobile authentication with JWTs.

### Syntax Rules and Structure

| Aspect | Session (`express-session`) | Stateless JWT |
|--------|---------------------------|---------------|
| State storage | Server-side (memory, Redis) | Client-side (token) |
| Cookie | Session ID | JWT (or none, if in-memory) |
| Logout | Immediate (destroy session) | Requires token blacklist or short expiry |
| Scaling | Requires shared session store | Horizontally scalable |
| Cross-domain | Limited by cookie domain | Works across domains |
| Mobile apps | Awkward (cookie handling) | Natural (token in header) |
| CSRF risk | Yes (cookie-based) | No (if not in cookie) |

**Implementation — Session-based:**
```js
const session = require('express-session');
const RedisStore = require('connect-redis')(session);

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: true,
    httpOnly: true,
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000
  }
}));
app.use(passport.initialize());
app.use(passport.session());
```

**Implementation — Stateless JWT:**
```js
// After successful social login, issue a JWT instead of a session
const jwt = require('jsonwebtoken');

app.get('/auth/google/callback',
  passport.authenticate('google', { session: false }),
  (req, res) => {
    const token = jwt.sign(
      { sub: req.user.id, email: req.user.email },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );
    res.json({ accessToken: token });
  }
);

// JWT verification middleware
function verifyJWT(req, res, next) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token' });
  }
  try {
    req.user = jwt.verify(authHeader.split(' ')[1], process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
}
```

**Rules:**
- Use sessions for server-rendered apps where you control the client and need immediate revocation.
- Use JWTs for distributed microservices, mobile apps, and cross-domain authentication.
- For social login with JWTs, pass `{ session: false }` to `passport.authenticate()`.
- Always store JWTs in HTTP-only cookies or memory, never in `localStorage`.
- Redis is recommended for session stores in production to avoid memory leaks and enable multi-process sharing.

#### Annotated Code Example — Hybrid Approach (Session for OAuth, JWT for API)

```js
// hybrid-auth.js
const express = require('express');
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;
const session = require('express-session');
const jwt = require('jsonwebtoken');
const app = express();

// Session is needed for the OAuth flow (state, PKCE)
app.use(session({ secret: 'oauth-session', resave: false, saveUninitialized: false }));
app.use(passport.initialize());
app.use(passport.session());

passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: 'http://localhost:3000/auth/google/callback'
}, (accessToken, refreshToken, profile, done) => {
  return done(null, { id: profile.id, name: profile.displayName });
}));

passport.serializeUser((user, done) => done(null, user));
passport.deserializeUser((user, done) => done(null, user));

// OAuth flow uses session for state management
app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] }));

// Callback issues a JWT instead of relying on session for API access
app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    const accessToken = jwt.sign(
      { sub: req.user.id, name: req.user.name },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );
    res.json({ accessToken, message: 'Use this token for API calls' });
  }
);

// API routes use stateless JWT
function requireJWT(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'Token required' });
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
}

app.get('/api/profile', requireJWT, (req, res) => {
  res.json({ user: req.user });
});

app.listen(3000, () => console.log('Hybrid auth on 3000'));
```

**Expected Output (for `GET /auth/google/callback` after authentication):**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "message": "Use this token for API calls"
}
```

**Expected Output (for `GET /api/profile` with `Authorization: Bearer <token>`):**
```json
{
  "user": {
    "sub": "1234567890",
    "name": "Alice Johnson",
    "iat": 1736996400,
    "exp": 1736997300
  }
}
```

**Why this output:** The OAuth flow uses a session to maintain state during the redirect dance (the `state` parameter and PKCE verifier must be stored somewhere). After successful authentication, the callback issues a JWT containing the user's identity. API routes then use this stateless JWT for authentication, avoiding server-side session lookups. This hybrid approach is common in modern applications: session for the OAuth dance, JWT for API access.

### Real-World Cases

- **Server-rendered web apps:** Sessions with `express-session` and Redis.
- **SPAs with separate API:** JWT issued after OAuth callback, stored in memory.
- **Mobile apps:** JWT with refresh tokens, no cookies.
- **Microservices:** JWT for inter-service authentication.
- **PCI-DSS compliance:** Server-side sessions with immediate logout capability.

---

## References

- Passport.js Documentation: Sessions — https://www.passportjs.org/concepts/authentication/sessions/
- Passport.js: passport-google-oauth20 — https://www.passportjs.org/packages/passport-google-oauth20/
- Passport.js: passport-github2 — https://www.passportjs.org/packages/passport-github2/
- passport-azure-ad (npm) — https://www.npmjs.com/package/passport-azure-ad
- Auth0: Add Login to Your Express Application — https://auth0.com/docs/quickstart/webapp/express
- Auth0: Protect Your Express.js API — https://auth0.com/docs/quickstart/backend/express
- Clerk: Express SDK Documentation — https://clerk.com/docs/references/express/overview
- Clerk: Upgrade from Node SDK to Express SDK — https://clerk.com/docs/guides/development/upgrading/upgrade-guides/node-to-express
- firebase-auth-express (npm) — https://www.npmjs.com/package/@my-f-startup/firebase-auth-express
- Firebase Auth Token Verification Middleware — https://github.com/OneUptime/blog/tree/master/posts/2026-02-17-firebase-auth-token-verification-express-middleware
- Express Session vs JWT — https://github.com/srivtx/js-miden/blob/main/docs_js_backend/04-authentication-authorization/README.md
- Microsoft: Signing Key Rollover with passport-azure-ad — https://learn.microsoft.com/en-us/entra/identity-platform/signing-key-rollover
- Passport.js Official Documentation — https://www.passportjs.org/docs/
- express-session (npm) — https://www.npmjs.com/package/express-session
- connect-redis (npm) — https://www.npmjs.com/package/connect-redis