# Passport.js — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Passport.js is authentication middleware for Node.js designed to serve a singular purpose: authenticating requests, while delegating unrelated details such as data access to the application.

**Technical Definition**
Passport is an Express-compatible authentication framework that uses **strategies**—self-contained modules that implement specific authentication mechanisms—to verify requests. It supports over 500 strategies, including local username/password, JWT, OAuth 1.0a/2.0, and OpenID. Passport cleanly encapsulates authentication logic while delegating user persistence and data access to the application, making it highly modular and easy to integrate into existing Express applications. For session-based authentication, Passport integrates with `express-session` and relies on `serializeUser`/`deserializeUser` functions to maintain login state across requests.

**Beginner-Friendly Explanation**
Passport.js is a toolkit that handles the messy parts of logging users in. Instead of writing separate login code for passwords, Google, GitHub, and API tokens, you plug in “strategies”—one for each method—and Passport takes care of the flow. It does not store your users or decide what they can do; it only answers the question “Who is making this request?” and hands the answer to your app.

---

### Key Characteristics

- **Strategy-based architecture** — Each authentication mechanism is a pluggable strategy, from `passport-local` for username/password to `passport-google-oauth20` for Google login.
- **Middleware-native** — `passport.authenticate()` is Express middleware that authenticates a request and either populates `req.user` and calls `next()`, or ends the cycle with a `401` (or a redirect for session-based web apps).
- **Session-optional** — Works with `express-session` for traditional web apps or with `{ session: false }` for stateless JWT-based APIs.
- **Verify-callback-driven** — Every strategy delegates credential verification to an application-supplied verify callback, keeping authentication logic under the developer's control.
- **Serialization-controlled** — The application decides exactly what minimal data is stored in the session and how to reconstruct the full user object.

---

### Prerequisites

- Node.js 18 or higher and an Express application initialized with `const app = express()`.
- Core packages: `passport`, a strategy package (e.g., `passport-local`, `passport-jwt`, `passport-google-oauth20`), and for session-based auth, `express-session`.
- For password hashing: `bcrypt` or `argon2`.
- For OAuth strategies: client ID, client secret, and callback URL from the provider.

---

### Related Programming Areas

- **Session authentication** — Passport's session integration relies on `express-session` and cookie-based state.
- **Token authentication** — JWT strategies enable stateless authentication for APIs.
- **OAuth 2.0 / OpenID Connect** — Social strategies implement delegated authentication.
- **Authorization** — Populated `req.user` drives downstream access-control decisions.
- **TypeScript** — Request augmentation provides typed `req.user` in TS applications.

---

## Core Concept 1: Passport.js

### Definitions

**Core Definition**
Passport.js is the library itself—the authentication middleware that coordinates strategies, manages login sessions, and populates `req.user` on successful authentication.

**Technical Definition**
Passport exports a singleton `passport` object with methods for configuring strategies (`passport.use()`), authenticating requests (`passport.authenticate()`), and managing sessions (`passport.initialize()`, `passport.session()`, `passport.serializeUser()`, `passport.deserializeUser()`). It is framework-agnostic but commonly used with Express, where it runs as middleware.

**Beginner-Friendly Explanation**
Passport.js is the main package. You import it, configure it with one or more strategies, and then use its middleware to authenticate requests.

---

### Purposes

- To provide a unified authentication middleware that supports any combination of strategies without ad-hoc glue code.
- To decouple authentication from application logic, keeping concerns separated.
- To handle strategy selection, credential verification, and session serialization consistently.
- To populate `req.user` for downstream authorization and personalization.

---

### Syntax Rules and Structure

#### Complete General Syntax: Initialization

```javascript
const passport = require('passport');

app.use(passport.initialize()); // always
app.use(passport.session());    // only for session-based auth
```

**Component Breakdown**

| Method | Role |
|--------|------|
| `passport.initialize()` | Initializes Passport middleware; required in all setups |
| `passport.session()` | Enables session-based login persistence; requires `express-session` |

#### Syntax Rules

- `passport.initialize()` must be mounted before routes and before `passport.session()`.
- `passport.session()` should be used only when session-based authentication is required; stateless API routes should use `{ session: false }`.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Minimal Passport Setup (No Strategy)

```javascript
const express = require('express');
const passport = require('passport');
const session = require('express-session');
const app = express();

app.use(session({ secret: process.env.SESSION_SECRET, resave: false, saveUninitialized: false }));
app.use(passport.initialize());
app.use(passport.session());

app.listen(3000);
```

**Expected Output**

- Passport middleware is active; `req.user` is available if a strategy authenticates a request.

---

### Real-World Cases

- **Multi-strategy applications** — Supporting local login, Google, GitHub, and JWT in the same app.
- **Progressive migration** — Adding Passport to an existing Express app without rewriting authentication from scratch.

---

## Core Concept 2: Strategies

### Definitions

**Core Definition**
A strategy is a self-contained authentication module that implements a specific mechanism—such as verifying a password, validating a JWT, or exchanging an OAuth code—and reports the result to Passport via a `done` callback.

**Technical Definition**
Each strategy package exports a `Strategy` constructor. The application instantiates it with configuration options and a **verify callback** `(…credentials, done)`. The verify callback locates or creates the user and calls `done(null, user)` on success, `done(null, false, info)` on authentication failure, or `done(err)` on error. Strategies are registered with `passport.use(strategy)` and used by name in `passport.authenticate('name')`.

**Beginner-Friendly Explanation**
A strategy is one method of logging in. Passport has a strategy for checking passwords, one for checking tokens, and one for each social provider. You pick the strategies you need, configure each one once, and then use them by name in your routes.

---

### Purposes

- To isolate the mechanics of each authentication method behind a uniform interface.
- To allow an application to support multiple login methods without conditional branching.
- To give the application full control over user lookup and credential verification via the verify callback.
- To enable stateless API auth (JWT) and stateful web auth (OAuth) with the same framework.

---

### Syntax Rules and Structure

#### Complete General Syntax: Registering a Strategy

```javascript
passport.use(new Strategy(options, verifyCallback));
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `Strategy` | The strategy constructor from a `passport-*` package |
| `options` | Strategy-specific configuration (fields, keys, URLs) |
| `verifyCallback` | Application function that verifies credentials and calls `done` |

#### Syntax Rules

- `done(null, user)` — authentication succeeded; `user` is passed to `serializeUser` (or `req.user` for stateless).
- `done(null, false, info)` — authentication failed; `info` carries a message.
- `done(err)` — an error occurred; forwarded to Express error handling.
- Strategy names must be unique per Passport instance.

---

### Sub-Concept: Local Strategy

#### Definitions

**Core Definition**
The local strategy authenticates users with a username (or email) and password submitted in the request body.

**Technical Definition**
`passport-local` reads `username` and `password` from `req.body` by default; `usernameField` and `passwordField` options customize the field names. The verify callback typically looks up the user, compares the password hash with `bcrypt.compare()`, and calls `done(null, user)` on success.

**Beginner-Friendly Explanation**
The local strategy is the classic “email and password” login. It checks credentials against your database and logs the user in if they match.

---

#### Purposes

- To authenticate users against application-managed credentials.
- To integrate password verification (bcrypt/argon2) into the Passport pipeline.
- To support traditional form-based login alongside social and token strategies.

---

#### Syntax Rules and Structure

```javascript
passport.use(new LocalStrategy({ usernameField: 'email' }, async (email, password, done) => {
  const user = await User.findOne({ email });
  if (!user) return done(null, false, { message: 'Invalid credentials' });
  const match = await bcrypt.compare(password, user.password);
  if (!match) return done(null, false, { message: 'Invalid credentials' });
  return done(null, user);
}));
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `usernameField: 'email'` | Maps the email field to the username parameter |
| Verify callback | Looks up user and compares password |
| `done(null, user)` | Signals success |

---

#### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

##### Example 1: Local Strategy Login

```javascript
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const bcrypt = require('bcrypt');

passport.use(new LocalStrategy(
  { usernameField: 'email' },
  async (email, password, done) => {
    try {
      const user = await User.findOne({ email });
      if (!user) return done(null, false, { message: 'Invalid email or password' });
      const isValid = await bcrypt.compare(password, user.password);
      if (!isValid) return done(null, false, { message: 'Invalid email or password' });
      return done(null, user);
    } catch (err) {
      return done(err);
    }
  }
));

app.post('/login', passport.authenticate('local', {
  successRedirect: '/dashboard',
  failureRedirect: '/login',
  failureMessage: true
}));
```

**Expected Output**

```
POST /login  (valid credentials)  → 302 /dashboard, req.user populated
POST /login  (invalid)            → 302 /login, req.session.messages set
```

**Why This Output Occurs**

- The local strategy verifies the password with bcrypt; success serializes the user and redirects, failure redirects back with a message.

---

#### Real-World Cases

- **Traditional web login** — Email/password forms on server-rendered sites.
- **Hybrid applications** — Local login as a fallback alongside social login.

---

### Sub-Concept: JWT Strategy

#### Definitions

**Core Definition**
The JWT strategy authenticates requests by extracting a JSON Web Token from the request, verifying its signature and claims, and loading the associated user.

**Technical Definition**
`passport-jwt` extracts the token via configurable extractors—most commonly `ExtractJwt.fromAuthHeaderAsBearerToken()`. The strategy verifies the token with a secret or public key, then passes the decoded payload to the verify callback, which looks up the user and calls `done(null, user)`. JWT authentication is **stateless**: it must be used with `{ session: false }` so Passport does not attempt to serialize the user into a session.

**Beginner-Friendly Explanation**
The JWT strategy is for APIs. It reads the token from the `Authorization` header, checks that it is valid, and then loads the user. No session is created.

---

#### Purposes

- To enable stateless API authentication without server-side sessions.
- To verify token signatures and claims on every request.
- To support mobile and single-page application clients that send bearer tokens.
- To integrate token-based auth with the same `req.user` contract as session-based auth.

---

#### Syntax Rules and Structure

```javascript
const JwtStrategy = require('passport-jwt').Strategy;
const { ExtractJwt } = require('passport-jwt');

passport.use(new JwtStrategy({
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: process.env.JWT_SECRET,
  algorithms: ['HS256'],
  issuer: 'my-api',
  audience: 'my-client'
}, async (payload, done) => {
  const user = await User.findById(payload.sub);
  if (!user) return done(null, false);
  return done(null, user);
}));

app.get('/api/protected',
  passport.authenticate('jwt', { session: false }),
  (req, res) => res.json({ user: req.user })
);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `jwtFromRequest` | Extracts the token from the `Authorization: Bearer` header |
| `secretOrKey` | Secret (HMAC) or public key (RSA/ECDSA) |
| `algorithms` | Explicit algorithm allowlist |
| `{ session: false }` | Disables session serialization for stateless auth |

---

#### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

##### Example 1: Stateless JWT Route

```javascript
app.get('/api/me',
  passport.authenticate('jwt', { session: false }),
  (req, res) => res.json({ id: req.user.id, email: req.user.email })
);
```

**Expected Output**

```
GET /api/me  (Authorization: Bearer <valid-jwt>)  → 200 { "id": 42, "email": "alice@example.com" }
GET /api/me  (no token)                          → 401 Unauthorized
```

**Why This Output Occurs**

- Passport extracts and verifies the token, loads the user, and `{ session: false }` prevents session creation.

---

#### Real-World Cases

- **REST APIs** — Stateless authentication for microservices and mobile clients.
- **Single-page applications** — Bearer tokens obtained after login.

---

### Sub-Concept: OAuth (Generic)

#### Definitions

**Core Definition**
OAuth strategies implement delegated authentication, where an external provider authenticates the user and Passport receives a profile that the application maps to a local user.

**Technical Definition**
OAuth 2.0 strategies (`passport-google-oauth20`, `passport-facebook`, `passport-github`) follow a common pattern: configure `clientID`, `clientSecret`, and `callbackURL`; the verify callback receives `(accessToken, refreshToken, profile, done)` and uses `profile` to find or create a local user. Twitter uses OAuth 1.0a, requiring `consumerKey` and `consumerSecret` instead.

**Beginner-Friendly Explanation**
OAuth strategies let users log in with an account they already have—like Google or GitHub. The provider confirms who they are, and Passport gives your app their profile so you can create or link a local account.

---

#### Purposes

- To delegate authentication to trusted identity providers.
- To reduce friction by letting users reuse existing accounts.
- To obtain verified profile data (email, name, avatar) from the provider.
- To support single sign-on (SSO) across applications.

---

#### Syntax Rules and Structure

```javascript
passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: '/auth/google/callback'
}, async (accessToken, refreshToken, profile, done) => {
  let user = await User.findOne({ googleId: profile.id });
  if (!user) user = await User.create({ googleId: profile.id, email: profile.emails[0].value });
  return done(null, user);
}));
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `clientID` / `clientSecret` | Provider credentials |
| `callbackURL` | Redirect URI registered with the provider |
| `profile` | Normalized provider profile object |
| Upsert logic | Finds or creates the local user |

---

### Sub-Concept: Google Strategy

#### Definitions

**Core Definition**
The Google strategy authenticates users via Google OAuth 2.0, using the `passport-google-oauth20` package.

**Technical Definition**
Configuration requires a Google Cloud project with an OAuth consent screen and OAuth 2.0 client credentials. The callback URL must match exactly in the Google Cloud Console, the environment variable, and the strategy configuration. The strategy returns a profile containing `id`, `displayName`, and `emails`.

**Beginner-Friendly Explanation**
Google login lets users sign in with their Google account. You register your app in Google Cloud, get a client ID and secret, and Passport handles the redirect flow.

---

#### Syntax Rules and Structure

```javascript
passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: '/auth/google/callback'
}, (accessToken, refreshToken, profile, done) => {
  return done(null, profile);
}));

app.get('/auth/google', passport.authenticate('google', { scope: ['profile', 'email'] }));
app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => res.redirect('/dashboard')
);
```

**Expected Output**

- Unauthenticated users are redirected to Google; on consent, the callback authenticates and redirects to the dashboard.

---

### Sub-Concept: Facebook Strategy

#### Definitions

**Core Definition**
The Facebook strategy authenticates users via Facebook OAuth 2.0 using the `passport-facebook` package.

**Technical Definition**
Configuration requires a Facebook app with `clientID`, `clientSecret`, and `callbackURL`. Since 2018, access to most profile fields beyond basic profile and email requires app review. The profile includes `id`, `displayName`, `emails`, and `photos`.

**Beginner-Friendly Explanation**
Facebook login works like Google login but with Facebook credentials.

---

#### Syntax Rules and Structure

```javascript
passport.use(new FacebookStrategy({
  clientID: process.env.FACEBOOK_APP_ID,
  clientSecret: process.env.FACEBOOK_APP_SECRET,
  callbackURL: '/auth/facebook/callback',
  profileFields: ['id', 'displayName', 'emails']
}, (accessToken, refreshToken, profile, done) => done(null, profile)));
```

---

### Sub-Concept: GitHub Strategy

#### Definitions

**Core Definition**
The GitHub strategy authenticates users via GitHub OAuth 2.0 using the `passport-github` package.

**Technical Definition**
Configuration requires `clientID`, `clientSecret`, and `callbackURL`. The profile includes `id`, `username`, `displayName`, and `emails`.

**Beginner-Friendly Explanation**
GitHub login is popular for developer tools and technical audiences.

---

#### Syntax Rules and Structure

```javascript
passport.use(new GitHubStrategy({
  clientID: process.env.GITHUB_CLIENT_ID,
  clientSecret: process.env.GITHUB_CLIENT_SECRET,
  callbackURL: '/auth/github/callback'
}, (accessToken, refreshToken, profile, done) => done(null, profile)));
```

---

### Sub-Concept: Twitter Strategy

#### Definitions

**Core Definition**
The Twitter strategy authenticates users via OAuth 1.0a using the `passport-twitter` package.

**Technical Definition**
Unlike the OAuth 2.0 strategies, Twitter uses OAuth 1.0a, which requires `consumerKey` and `consumerSecret` (not `clientID`/`clientSecret`) and cryptographically signs every request. The verify callback receives `(token, tokenSecret, profile, done)`.

**Beginner-Friendly Explanation**
Twitter login uses an older OAuth version, so the option names differ, but the overall flow is the same.

---

#### Syntax Rules and Structure

```javascript
passport.use(new TwitterStrategy({
  consumerKey: process.env.TWITTER_CONSUMER_KEY,
  consumerSecret: process.env.TWITTER_CONSUMER_SECRET,
  callbackURL: '/auth/twitter/callback'
}, (token, tokenSecret, profile, done) => done(null, profile)));
```

---

### Strategy Comparison

| Strategy | Package | Protocol | Key Options | Verify Signature |
|----------|---------|----------|-------------|------------------|
| Local | `passport-local` | N/A | `usernameField`, `passwordField` | `(username, password, done)` |
| JWT | `passport-jwt` | N/A | `jwtFromRequest`, `secretOrKey` | `(payload, done)` |
| Google | `passport-google-oauth20` | OAuth 2.0 | `clientID`, `clientSecret`, `callbackURL` | `(accessToken, refreshToken, profile, done)` |
| Facebook | `passport-facebook` | OAuth 2.0 | `clientID`, `clientSecret`, `callbackURL` | `(accessToken, refreshToken, profile, done)` |
| GitHub | `passport-github` | OAuth 2.0 | `clientID`, `clientSecret`, `callbackURL` | `(accessToken, refreshToken, profile, done)` |
| Twitter | `passport-twitter` | OAuth 1.0a | `consumerKey`, `consumerSecret`, `callbackURL` | `(token, tokenSecret, profile, done)` |

---

## Core Concept 3: `passport.authenticate()`

### Definitions

**Core Definition**
`passport.authenticate()` is the middleware function that invokes a named strategy to authenticate a request, and on success populates `req.user` and continues the middleware chain.

**Technical Definition**
`passport.authenticate(strategy, options, callback)` returns middleware. By default, on success it sets `req.user`, establishes a login session (when sessions are enabled), and calls `next()`. On failure, it sends `401 Unauthorized` and ends the cycle. The `failureRedirect` option changes this to a `302` redirect, and a custom callback can override the default behavior entirely.

**Beginner-Friendly Explanation**
`passport.authenticate()` is the gatekeeper. You tell it which strategy to use, and it either lets the request through (with `req.user` set) or stops it.

---

### Purposes

- To invoke a specific strategy and apply its authentication result to the request.
- To populate `req.user` for downstream handlers.
- To control failure behavior (401 vs. redirect vs. custom callback).
- To customize success and failure handling with a callback when default behavior is insufficient.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
passport.authenticate(strategyName, options, callback?);
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `strategyName` | The name registered with `passport.use()` |
| `options` | `session`, `failureRedirect`, `failureMessage`, `successRedirect` |
| `callback` | Optional custom `(err, user, info)` handler |

#### Syntax Rules

- Use `{ session: false }` for stateless JWT authentication.
- `failureRedirect` is appropriate for session-based web apps; omit it for REST APIs to get `401`.
- A custom callback must call `req.logIn()` manually to establish a session.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Default, Redirect, and Custom Callback

```javascript
// Default (REST): 401 on failure
app.get('/api/profile', passport.authenticate('jwt', { session: false }), handler);

// Redirect (web): 302 on failure
app.post('/login', passport.authenticate('local', {
  successRedirect: '/dashboard',
  failureRedirect: '/login'
}));

// Custom callback for JSON API
app.post('/api/login', (req, res, next) => {
  passport.authenticate('local', (err, user, info) => {
    if (err) return next(err);
    if (!user) return res.status(401).json({ error: info.message });
    req.logIn(user, (err) => {
      if (err) return next(err);
      res.json({ id: user.id, email: user.email });
    });
  })(req, res, next);
});
```

**Expected Output**

```
GET /api/profile  (no token)       → 401 Unauthorized
POST /login       (invalid)        → 302 /login
POST /api/login   (invalid)        → 401 { "error": "Invalid email or password" }
```

**Why This Output Occurs**

- The default sends `401`; `failureRedirect` sends `302`; the custom callback controls the response format.

---

### Real-World Cases

- **REST APIs** — JWT strategy with default `401` behavior.
- **Traditional web apps** — Local strategy with `failureRedirect` to the login page.
- **Hybrid APIs** — Custom callbacks that return JSON tokens instead of redirecting.

---

## Core Concept 4: Serialization

### Definitions

**Core Definition**
Serialization is the process of determining what minimal user information is stored in the session after a successful login, so that subsequent requests can identify the user without re-authentication.

**Technical Definition**
Passport calls `passport.serializeUser(user, done)` after successful authentication, just before the session is created. The function calls `done(null, data)`, where `data` is stored in the session (typically the user ID or a small object). Best practice is to store only a stable identifier (e.g., `user.id`) to keep session size small.

**Beginner-Friendly Explanation**
Serialization decides what “sticky note” Passport puts in the session cookie to remember the user. You want that note to be small—usually just the user ID—not the entire user record.

---

### Purposes

- To store the minimal information needed to identify the user on future requests.
- To keep session cookies small and efficient.
- To avoid storing sensitive or large data in the session.
- To define the contract that `deserializeUser` will use to reconstruct the user.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
passport.serializeUser((user, done) => {
  done(null, user.id);
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `user` | The authenticated user object from the strategy's `done` callback |
| `done(null, user.id)` | Stores the user ID in the session |

#### Syntax Rules

- Call `done(null, data)` with the minimal identifier.
- Never call `done(null, user)` with the entire user object unless you intentionally want it in the session.
- Serialization runs only after login, just before session creation.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Storing Only the User ID

```javascript
passport.serializeUser((user, done) => {
  done(null, user.id);
});
```

**Expected Output**

- The session contains only `{ passport: { user: 42 } }` rather than the full user record.

---

### Real-World Cases

- **Session-based web apps** — Minimal session data for scalable session stores.
- **Multi-tenant apps** — Storing tenant ID alongside user ID in the session.

---

## Core Concept 5: Deserialization

### Definitions

**Core Definition**
Deserialization is the process of reconstructing the full user object from the minimal data stored in the session on every authenticated request.

**Technical Definition**
`passport.deserializeUser(data, done)` is called on every request that has a session cookie. It receives the data stored by `serializeUser` (e.g., the user ID) and must call `done(null, user)` with the full user object, which Passport assigns to `req.user`. Because this runs on every authenticated request, performance matters: database lookups should be indexed, and caching may be appropriate.

**Beginner-Friendly Explanation**
Deserialization takes the sticky note from the session cookie and looks up the full user. It runs on every request, so it should be fast.

---

### Purposes

- To reconstruct `req.user` on every authenticated request.
- To keep session data minimal while still providing the full user object to handlers.
- To centralize user-loading logic and error handling.

---

### Syntax Rules and Structure

#### Complete General Syntax

```javascript
passport.deserializeUser(async (id, done) => {
  try {
    const user = await User.findById(id);
    done(null, user);
  } catch (err) {
    done(err);
  }
});
```

**Component Breakdown**

| Component | Role |
|-----------|------|
| `id` | The data stored by `serializeUser` |
| `User.findById(id)` | Loads the full user |
| `done(null, user)` | Populates `req.user` |

#### Syntax Rules

- Always handle errors and call `done(err)` on failure.
- Return `done(null, false)` if the user no longer exists, rather than throwing.
- Because this runs on every authenticated request, use efficient queries and consider caching.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Deserialization with Error Handling

```javascript
passport.deserializeUser(async (id, done) => {
  try {
    const user = await User.findById(id);
    if (!user) return done(null, false);
    done(null, user);
  } catch (err) {
    done(err);
  }
});
```

**Expected Output**

- Valid session → `req.user` is populated; deleted user → `req.user` is falsy and the request is treated as unauthenticated.

---

### Real-World Cases

- **Session-based APIs** — Reconstructing the user for each request.
- **Caching layers** — Caching user objects to reduce database load.

---

## Core Concept 6: Passport Best Practices

### Definitions

**Core Definition**
Passport best practices are the configuration and implementation patterns that keep authentication secure, maintainable, and correctly integrated with Express sessions and TypeScript.

**Technical Definition**
Best practices include: using `{ session: false }` for all API and JWT routes; separating `passport.initialize()` (always) from `passport.session()` (only for session-based auth); storing only the user ID in the session; handling errors in `deserializeUser`; augmenting `Express.User` in TypeScript for typed `req.user`; splitting strategies into individual files; and using `failureMessage: true` to surface authentication errors in session-based flows.

**Beginner-Friendly Explanation**
Best practices are the habits that keep Passport safe and predictable. Do not use sessions for APIs, keep session data tiny, handle errors in deserialization, and keep each strategy in its own file so the code stays organized.

---

### Purposes

- To prevent session-related bugs in stateless APIs.
- To keep session data minimal and session stores efficient.
- To ensure errors in deserialization are handled rather than crashing the app.
- To maintain clean, testable strategy configuration.

---

### Syntax Rules and Structure

#### Complete General Syntax: Hardened Passport Setup

```javascript
// 1. Session middleware (only if needed)
app.use(session({ secret: process.env.SESSION_SECRET, resave: false, saveUninitialized: false }));

// 2. Initialize Passport, then session support
app.use(passport.initialize());
app.use(passport.session()); // only for session-based auth

// 3. Minimal serialization
passport.serializeUser((user, done) => done(null, user.id));

// 4. Safe deserialization
passport.deserializeUser(async (id, done) => {
  try {
    const user = await User.findById(id);
    done(null, user || false);
  } catch (err) {
    done(err);
  }
});

// 5. Stateless JWT routes
app.get('/api/me', passport.authenticate('jwt', { session: false }), handler);
```

**Component Breakdown**

| Practice | Implementation |
|----------|----------------|
| No sessions for APIs | `{ session: false }` |
| Correct middleware order | `initialize()` before `session()` |
| Minimal session data | `done(null, user.id)` |
| Safe deserialization | `try/catch` with `done(err)` |
| Typed user | Augment `Express.User` in TypeScript |

#### Syntax Rules

- **Never** use `passport.session()` for JWT-only APIs.
- **Always** call `passport.initialize()` before `passport.session()`.
- **Store only the user ID** in the session; let `deserializeUser` fetch the full user.
- **Handle errors** in `deserializeUser` and return `done(null, false)` for missing users.
- **Augment `Express.User`** in TypeScript to avoid `unknown` types on `req.user`.
- **Separate strategy files** for maintainability.

#### Constraints and Limitations

- `deserializeUser` runs on every authenticated request; caching may be necessary for high-traffic apps.
- Forgetting `{ session: false }` on JWT routes causes Passport to attempt session serialization, which can crash or behave unexpectedly.

---

### Multiple Annotated Complete Step-by-Step Code Examples and Expected Outputs

#### Example 1: Complete Hardened Passport Pipeline

```javascript
const express = require('express');
const passport = require('passport');
const session = require('express-session');
const LocalStrategy = require('passport-local').Strategy;
const JwtStrategy = require('passport-jwt').Strategy;
const { ExtractJwt } = require('passport-jwt');
const bcrypt = require('bcrypt');
const app = express();

app.use(express.json());
app.use(session({ secret: process.env.SESSION_SECRET, resave: false, saveUninitialized: false }));
app.use(passport.initialize());
app.use(passport.session());

// Local strategy
passport.use(new LocalStrategy({ usernameField: 'email' }, async (email, password, done) => {
  const user = await User.findOne({ email });
  if (!user) return done(null, false, { message: 'Invalid credentials' });
  const ok = await bcrypt.compare(password, user.password);
  if (!ok) return done(null, false, { message: 'Invalid credentials' });
  return done(null, user);
}));

// JWT strategy
passport.use(new JwtStrategy({
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: process.env.JWT_SECRET,
  algorithms: ['HS256']
}, async (payload, done) => {
  const user = await User.findById(payload.sub);
  if (!user) return done(null, false);
  return done(null, user);
}));

// Serialization
passport.serializeUser((user, done) => done(null, user.id));
passport.deserializeUser(async (id, done) => {
  try {
    const user = await User.findById(id);
    done(null, user || false);
  } catch (err) {
    done(err);
  }
});

// Routes
app.post('/login', passport.authenticate('local', { successRedirect: '/dashboard', failureRedirect: '/login' }));
app.get('/api/me', passport.authenticate('jwt', { session: false }), (req, res) => res.json(req.user));

app.listen(3000);
```

**Expected Output**

```
POST /login  (valid)  → 302 /dashboard (session established)
GET /api/me  (valid Bearer)  → 200 { user data } (no session created)
```

**Why This Output Occurs**

- The local strategy uses sessions; the JWT strategy explicitly disables sessions, and deserialization safely handles missing users.

---

### Real-World Cases

- **Production APIs** — JWT strategy with `{ session: false }` and hardened verification.
- **Hybrid web/API apps** — Local + session for the web, JWT for the API.
- **Large teams** — Strategy-per-file organization and typed `req.user` for maintainability.

---

## References

- Passport.js Documentation: Overview — https://www.passportjs.org/concepts/authentication/downloads/html/
- Passport.js Documentation: Sessions — http://www.passportjs.org/concepts/authentication/sessions/
- How to Use Passport.js for Authentication in Node.js — https://oneuptime.com/blog/post/2026-01-22-nodejs-passport-authentication/view
- Passport.js with TypeScript — Authentication Strategies Guide 2026 — https://webcoderspeed.com/blog/typescript-backend/32-passport-guide
- How to use Passport.js in Node.js · CoreUI — https://coreui.io/answers/how-to-use-passportjs-in-nodejs/
- Mastering Authentication in Express with Passport.js Strategies — https://leapcell.io/blog/mastering-authentication-in-express-with-passport-js-strategies
- passport-google-oauth20 vs passport-facebook vs passport-github vs passport-twitter — https://npm-compare.com/passport-facebook,passport-github,passport-google-oauth20,passport-twitter
- Google Authentication in Express.js with TypeScript, Mongoose & JWT — https://nheroop.github.io/posts/google-auth-express-ts/