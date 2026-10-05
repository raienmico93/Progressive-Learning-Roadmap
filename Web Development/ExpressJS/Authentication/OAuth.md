# OAuth & Modern Standards — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** OAuth 2.1 is the consolidated authorization framework that merges OAuth 2.0 (RFC 6749) with its security best practices (RFC 9700), PKCE (RFC 7636), native app guidance (RFC 8252), and browser-based app guidance into a single, streamlined specification that removes insecure legacy flows and mandates modern security controls.

**Technical Definition:** OAuth 2.1 is an IETF Internet-Draft that consolidates OAuth 2.0 and its extensions into a single specification. It defines only two grant types — Authorization Code with PKCE and Client Credentials — omits the Implicit and Resource Owner Password Credentials grants, requires exact redirect URI matching, mandates PKCE for all authorization code clients, and requires refresh tokens for public clients to be either sender-constrained or one-time use.

**Beginner-Friendly Explanation:** OAuth is a system that lets you give an app permission to access your data on another service without sharing your password. For example, when a photo-printing app asks to access your Google Photos, OAuth lets you approve that access without giving the app your Google password. OAuth 2.1 is the updated version that fixes security problems found in the original OAuth 2.0 — it removes old, unsafe methods and requires modern protections like PKCE.

### Key Characteristics

- **Delegated authorization:** Allows users to grant limited access to their resources without sharing credentials.
- **Token-based:** Uses access tokens (short-lived) and refresh tokens (long-lived) to manage sessions.
- **PKCE mandatory:** All authorization code clients must use Proof Key for Code Exchange.
- **Two grant types only:** Authorization Code with PKCE and Client Credentials.
- **Exact redirect URI matching:** Eliminates wildcard-based attacks.
- **Refresh token rotation:** Public clients must use one-time-use or sender-constrained refresh tokens.
- **No tokens in URLs:** Bearer tokens must never appear in query strings.

### Prerequisites

- **Basic understanding of HTTP:** Requests, responses, headers, and status codes.
- **Familiarity with web authentication concepts:** Sessions, cookies, and tokens.
- **Basic knowledge of cryptography:** Hashing (SHA-256) and random number generation.
- **Understanding of client-server architecture.**

### Related Programming Areas

- **OpenID Connect (OIDC):** Identity layer built on OAuth 2.0 for authentication.
- **API security:** Token-based access control for REST APIs.
- **Single-page applications (SPAs):** Browser-based apps using PKCE.
- **Mobile applications:** Native apps using PKCE and custom URI schemes.
- **Microservices:** Machine-to-machine communication using Client Credentials.
- **Zero Trust architecture:** Sender-constrained tokens and proof-of-possession.

### Core Concepts

1. **OAuth Concepts** — Roles, Scopes, and Grants.
2. **Authorization Flows** — Authorization Code Grant, Client Credentials, and deprecated grants.
3. **OAuth 2.1 Upgrades** — PKCE for all clients.
4. **Token Management** — Access tokens, refresh tokens, expiration, and rotation.
5. **Redirect URIs** — Validation, wildcard risks, and state parameters for CSRF protection.

---

## Core Concept 1: OAuth Concepts — Roles, Scopes, and Grants

### Definitions

**Core Definition:** OAuth defines four roles (Resource Owner, Client, Authorization Server, Resource Server) and uses scopes to limit access and grants to obtain tokens.

**Technical Definition:** OAuth 2.0 defines four roles: the **resource owner** (an entity capable of granting access to a protected resource, typically the end-user), the **resource server** (the server hosting protected resources), the **client** (an application making protected resource requests on behalf of the resource owner), and the **authorization server** (the server issuing access tokens after authenticating the resource owner and obtaining authorization). An **authorization grant** is a credential representing the resource owner's authorization, expressed using one of several grant types. A **scope** is a space-delimited list of case-sensitive strings that define the access range of an access token.

**Beginner-Friendly Explanation:** Think of OAuth like a hotel key card system. The **Resource Owner** is the hotel guest (you). The **Client** is the hotel app on your phone that wants to access your room. The **Authorization Server** is the front desk that verifies your identity and gives you a key card. The **Resource Server** is your actual hotel room. **Scopes** determine which doors your key card can open (room only, gym, pool, etc.). **Grants** are the different ways you can get that key card.

### Purposes

- To define the entities involved in delegated authorization.
- To limit access using scopes so clients only get the permissions they need.
- To standardise how authorization is granted and tokens are issued.
- To separate authentication from authorization.

### Syntax Rules and Structure

#### OAuth Roles

| Role | Description | Example |
|------|-------------|---------|
| Resource Owner | Entity granting access | End-user (you) |
| Client | Application requesting access | Photo-printing app |
| Authorization Server | Issues tokens | Google's OAuth server |
| Resource Server | Hosts protected resources | Google Photos API |

#### Scope Syntax

```
scope = scope-token *( SP scope-token )
scope-token = 1*( %x21 / %x23-5B / %x5D-7E )
```

| Component | Breakdown |
|-----------|-----------|
| `scope` | Space-delimited list of scope tokens. |
| `scope-token` | Case-sensitive string defining an access range. |

#### Grant Types in OAuth 2.1

| Grant Type | Use Case | PKCE Required |
|------------|----------|---------------|
| Authorization Code | User-facing apps | Yes (mandatory) |
| Client Credentials | Machine-to-machine | No |

#### Syntax Rules

- Scopes are space-delimited and case-sensitive.
- OAuth does not define scope values; the authorization server defines them.
- The authorization server may fully or partially ignore requested scopes.
- Confidential clients must authenticate with the authorization server.
- The client credentials grant MUST only be used by confidential clients.

#### Constraints and Limitations

- The Implicit grant and Resource Owner Password Credentials grant are omitted from OAuth 2.1.
- Bearer tokens must not be sent in query strings.
- Scopes alone do not provide fine-grained resource-level authorization.

### Annotated Code Example

```js
// oauth-roles-scopes.js
const express = require('express');
const app = express();

// Simulated OAuth 2.1 authorization server
const authServer = {
  scopes: {
    'read:profile': 'Read user profile',
    'read:email'  : 'Read user email',
    'write:posts' : 'Create and edit posts'
  },

  // Validate requested scopes
  validateScopes(requestedScopes) {
    const valid = Object.keys(this.scopes);
    return requestedScopes.filter(s => valid.includes(s));
  },

  // Issue access token with granted scopes
  issueToken(clientId, userId, scopes) {
    return {
      access_token: 'at_' + Math.random().toString(36).substring(2),
      token_type: 'Bearer',
      expires_in: 3600,
      scope: scopes.join(' ')
    };
  }
};

// Client requests authorization with specific scopes
app.get('/oauth/authorize', (req, res) => {
  const { client_id, scope, redirect_uri, state } = req.query;
  const requestedScopes = (scope || '').split(' ').filter(Boolean);
  const grantedScopes = authServer.validateScopes(requestedScopes);

  // In production: redirect to consent page
  res.json({
    client_id,
    requested: requestedScopes,
    granted: grantedScopes,
    message: 'User must approve these scopes'
  });
});

app.listen(3000, () => console.log('OAuth roles server on 3000'));
```

**Expected Output (for `GET /oauth/authorize?client_id=app1&scope=read:profile%20read:email&redirect_uri=https://app.com/cb&state=xyz`):**
```json
{
  "client_id": "app1",
  "requested": ["read:profile", "read:email"],
  "granted": ["read:profile", "read:email"],
  "message": "User must approve these scopes"
}
```

**Why this output:** The client requests two scopes (`read:profile` and `read:email`). The authorization server validates them against its registered scopes and confirms both are valid. In a real flow, the user would then see a consent screen listing these permissions before approving.

### Real-World Cases

- **Google APIs:** Scopes like `https://www.googleapis.com/auth/gmail.readonly` limit access to read-only Gmail.
- **GitHub OAuth:** Scopes like `repo`, `user`, `gist` control repository, profile, and gist access.
- **Slack OAuth:** Scopes like `channels:read`, `chat:write` control channel and message permissions.

---

## Core Concept 2: Authorization Flows

### Definitions

**Core Definition:** Authorization flows are the sequences of steps a client follows to obtain an access token from the authorization server, with different flows suited to different client types and use cases.

**Technical Definition:** OAuth 2.1 defines two grant types: **Authorization Code with PKCE** (for user-facing clients including web apps, SPAs, mobile, and native apps) and **Client Credentials** (for machine-to-machine communication where no user is involved). The Implicit grant and Resource Owner Password Credentials grant are omitted from OAuth 2.1 as they were deprecated in RFC 9700.

**Beginner-Friendly Explanation:** An authorization flow is like a recipe for getting permission. The **Authorization Code flow** is for apps where a user is present — you go to the authorization server, log in, approve the app, and get a temporary code that the app exchanges for a real token. The **Client Credentials flow** is for server-to-server communication — the server identifies itself and gets a token directly, with no user involved.

### Purposes

- To provide appropriate authorization mechanisms for different client types.
- To keep tokens out of the browser URL (unlike the deprecated Implicit flow).
- To enable secure machine-to-machine communication.
- To prevent credential sharing (unlike the deprecated Password grant).

---

### Sub-Feature 2.1: Authorization Code Grant with PKCE

#### Definitions

**Core Definition:** The Authorization Code grant with PKCE is the recommended flow for all user-facing clients, where the client receives an authorization code and exchanges it for tokens using a PKCE proof.

**Technical Definition:** In the Authorization Code flow, the client directs the resource owner's user-agent to the authorization endpoint. The authorization server authenticates the user and issues an authorization code, which the client exchanges at the token endpoint for an access token and (optionally) a refresh token. PKCE adds a `code_challenge` (sent with the authorization request) and a `code_verifier` (sent with the token request) to prove that the same client that initiated the flow is completing it.

**Beginner-Friendly Explanation:** The Authorization Code flow is like a two-step verification at a hotel: you check in at the front desk (authorization server) and get a claim ticket (authorization code). You then take that ticket to the bell desk (token endpoint) to get your room key (access token). PKCE adds a secret handshake so that even if someone steals your claim ticket, they can't get your room key without the secret.

#### Purposes

- To securely obtain access tokens for user-facing applications.
- To prevent authorization code interception attacks via PKCE.
- To support refresh tokens for long-lived sessions.
- To keep tokens out of browser history and referrer headers.

#### Syntax Rules and Structure

**Step 1: Generate PKCE Parameters**
```js
const verifier = base64url(crypto.randomBytes(32));
const challenge = base64url(sha256(verifier));
```
| Component | Breakdown |
|-----------|-----------|
| `code_verifier` | High-entropy random string, 43–128 characters. |
| `code_challenge` | `BASE64URL(SHA256(code_verifier))` for S256 method. |

**Step 2: Authorization Request**
```
GET /authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=REDIRECT_URI&
  scope=SCOPES&
  state=STATE&
  code_challenge=CHALLENGE&
  code_challenge_method=S256
```

**Step 3: Token Exchange**
```
POST /token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTH_CODE&
redirect_uri=REDIRECT_URI&
client_id=CLIENT_ID&
code_verifier=VERIFIER
```

#### Syntax Rules

- `code_challenge_method` defaults to `plain` if not specified; `S256` is Mandatory To Implement (MTI) on the server.
- The authorization code MUST expire shortly after issuance (maximum 10 minutes recommended) and MUST NOT be used more than once.
- In OAuth 2.1, the `redirect_uri` parameter is no longer sent in the token request; PKCE prevents code injection.
- The `state` parameter MUST be used for CSRF protection unless PKCE is relied upon.

#### Constraints and Limitations

- PKCE does not protect against all attacks; sender-constrained tokens (DPoP, mTLS) provide additional protection.
- The `plain` method should only be used when the client cannot perform SHA-256 hashing.

#### Annotated Code Example

```js
// authorization-code-pkce.js
const express = require('express');
const crypto = require('crypto');
const app = express();
app.use(express.json());

// In-memory store for demo (use session/database in production)
const sessions = new Map();

// Step 1: Generate PKCE pair
function generatePKCE() {
  const verifier = crypto.randomBytes(32).toString('base64url');
  const challenge = crypto
    .createHash('sha256')
    .update(verifier)
    .digest('base64url');
  return { verifier, challenge };
}

// Step 2: Initiate authorization request
app.get('/login', (req, res) => {
  const { verifier, challenge } = generatePKCE();
  const state = crypto.randomUUID();

  // Store verifier and state for later verification
  sessions.set(state, { verifier, createdAt: Date.now() });

  const authUrl = new URL('https://auth.example.com/authorize');
  authUrl.searchParams.set('response_type', 'code');
  authUrl.searchParams.set('client_id', 'my-client-id');
  authUrl.searchParams.set('redirect_uri', 'http://localhost:3000/callback');
  authUrl.searchParams.set('scope', 'openid profile email');
  authUrl.searchParams.set('state', state);
  authUrl.searchParams.set('code_challenge', challenge);
  authUrl.searchParams.set('code_challenge_method', 'S256');

  res.redirect(authUrl.toString());
});

// Step 3: Handle callback and exchange code for tokens
app.get('/callback', async (req, res) => {
  const { code, state } = req.query;
  const session = sessions.get(state);

  if (!session) {
    return res.status(400).json({ error: 'Invalid state parameter' });
  }

  // Verify state to prevent CSRF
  sessions.delete(state);

  // Exchange authorization code for tokens
  const tokenResponse = await fetch('https://auth.example.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code,
      redirect_uri: 'http://localhost:3000/callback',
      client_id: 'my-client-id',
      code_verifier: session.verifier  // PKCE proof
    })
  });

  const tokens = await tokenResponse.json();
  res.json({ message: 'Tokens received', tokens });
});

app.listen(3000, () => console.log('PKCE server on 3000'));
```

**Expected Output (for the redirect to the authorization server):**
```
https://auth.example.com/authorize?response_type=code&client_id=my-client-id&redirect_uri=http%3A%2F%2Flocalhost%3A3000%2Fcallback&scope=openid+profile+email&state=<uuid>&code_challenge=<hash>&code_challenge_method=S256
```

**Expected Output (for the token exchange):**
```json
{
  "message": "Tokens received",
  "tokens": {
    "access_token": "eyJhbGciOiJSUzI1NiIs...",
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_token": "def50200...",
    "scope": "openid profile email"
  }
}
```

**Why this output:** The client generates a PKCE verifier and challenge, stores the verifier in a session, and redirects the user to the authorization server with the challenge. After the user authenticates, the authorization server redirects back with an authorization code. The client then exchanges the code for tokens, including the `code_verifier` to prove it initiated the flow. The `state` parameter prevents CSRF attacks.

#### Real-World Cases

- **Google Sign-In:** Web and mobile apps use Authorization Code with PKCE.
- **Auth0/Okta:** All modern integrations use Authorization Code with PKCE.
- **Microsoft Entra ID:** SPAs and web apps use auth code flow with PKCE.

---

### Sub-Feature 2.2: Client Credentials Grant

#### Definitions

**Core Definition:** The Client Credentials grant is used for machine-to-machine communication where the client authenticates directly with its own credentials and receives an access token without user involvement.

**Technical Definition:** The client can request an access token using only its client credentials (or other supported means of authentication) when the client is requesting access to protected resources under its control, or those of another resource owner that have been previously arranged with the authorization server. The client credentials grant type MUST only be used by confidential clients.

**Beginner-Friendly Explanation:** The Client Credentials flow is like a server identifying itself with a username and password to get a key that lets it access another server. There's no user involved — it's just two servers talking to each other.

#### Purposes

- To enable secure machine-to-machine communication.
- To authenticate background services and daemons.
- To access APIs that don't require user context.

#### Syntax Rules and Structure

```
POST /token
Content-Type: application/x-www-form-urlencoded
Authorization: Basic BASE64(CLIENT_ID:CLIENT_SECRET)

grant_type=client_credentials&
scope=SCOPE
```

| Component | Breakdown |
|-----------|-----------|
| `grant_type` | Set to `client_credentials`. |
| `client_id` / `client_secret` | Sent via Basic Auth or request body. |
| `scope` | Optional; the requested access scope. |

#### Syntax Rules

- Confidential clients MUST authenticate with the authorization server.
- The authorization server MUST authenticate the client before issuing a token.
- The `scope` parameter is optional.

#### Annotated Code Example

```js
// client-credentials.js
const express = require('express');
const app = express();
app.use(express.json());

// Simulated token endpoint
app.post('/oauth/token', (req, res) => {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Basic ')) {
    return res.status(401).json({ error: 'invalid_client' });
  }

  // Decode Basic Auth credentials
  const credentials = Buffer.from(authHeader.split(' ')[1], 'base64')
    .toString()
    .split(':');
  const [clientId, clientSecret] = credentials;

  // Validate client credentials (simulated)
  if (clientId !== 'service-a' || clientSecret !== 'secret-123') {
    return res.status(401).json({ error: 'invalid_client' });
  }

  // Issue access token
  res.json({
    access_token: 'at_' + Math.random().toString(36).substring(2),
    token_type: 'Bearer',
    expires_in: 3600,
    scope: req.body.scope || 'read:data'
  });
});

app.listen(3000, () => console.log('Client credentials server on 3000'));
```

**Expected Output (for `POST /oauth/token` with `grant_type=client_credentials&scope=read:data`):**
```json
{
  "access_token": "at_abc123xyz",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "read:data"
}
```

**Why this output:** The client authenticates using Basic Auth with its client ID and secret. The authorization server validates the credentials and issues an access token with the requested scope. No user interaction is required.

#### Real-World Cases

- **Microservices:** Internal services authenticating to call each other.
- **CI/CD pipelines:** Automated systems accessing deployment APIs.
- **Data synchronization:** Background jobs syncing data between systems.

---

### Sub-Feature 2.3: Deprecated Grants — Implicit and Password

#### Definitions

**Core Definition:** The Implicit grant and Resource Owner Password Credentials (ROPC) grant are legacy OAuth 2.0 flows that are deprecated in OAuth 2.1 due to security vulnerabilities.

**Technical Definition:** The Implicit grant returned access tokens directly in the URL fragment, where they leak through browser history and referrers. The Resource Owner Password Credentials grant insecurely exposes the resource owner's credentials to the client. Both grants are omitted from OAuth 2.1.

**Beginner-Friendly Explanation:** The Implicit flow was like getting your room key in the lobby where everyone can see it — insecure. The Password flow was like giving your password to the front desk clerk so they can log in as you — also insecure. Both are no longer allowed in modern OAuth.

#### Purposes (Deprecated — Retained for Legacy Understanding)

- To understand why legacy systems may still use these flows.
- To plan migration away from these insecure grants.
- To recognise insecure implementations during security audits.

#### Annotated Code Example — Migration from Implicit to Auth Code + PKCE

```js
// Before (OAuth 2.0 Implicit — DEPRECATED)
// const authUrl = `/authorize?response_type=token&client_id=...`;
// Token arrives in URL fragment: https://app.com/cb#access_token=...

// After (OAuth 2.1 Authorization Code + PKCE)
const crypto = require('crypto');

function generatePKCE() {
  const verifier = crypto.randomBytes(32).toString('base64url');
  const challenge = crypto
    .createHash('sha256')
    .update(verifier)
    .digest('base64url');
  return { verifier, challenge };
}

const { verifier, challenge } = generatePKCE();
const authUrl = `/authorize?
  response_type=code&
  client_id=my-client&
  redirect_uri=https://app.com/callback&
  code_challenge=${challenge}&
  code_challenge_method=S256`;

// Store verifier securely for token exchange
sessionStorage.setItem('pkce_verifier', verifier);

console.log('Auth URL:', authUrl);
```

**Expected Output:**
```
Auth URL: /authorize?response_type=code&client_id=my-client&redirect_uri=https://app.com/callback&code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM&code_challenge_method=S256
```

**Why this output:** The migration replaces `response_type=token` (Implicit) with `response_type=code` and adds PKCE parameters. The access token is no longer exposed in the URL; instead, a short-lived code is returned and exchanged server-side.

### Real-World Cases

- **Legacy SPA migration:** Moving from Implicit to Auth Code + PKCE.
- **Third-party integrations:** Updating OAuth clients to comply with RFC 9700.
- **Security audits:** Identifying deprecated grant usage in existing applications.

---

## Core Concept 3: OAuth 2.1 Upgrades — PKCE for All Clients

### Definitions

**Core Definition:** PKCE (Proof Key for Code Exchange) is a security extension that binds the authorization code to the client that requested it, preventing code interception and injection attacks.

**Technical Definition:** PKCE requires the client to generate a random `code_verifier` and derive a `code_challenge` from it using a transformation (S256 or plain). The challenge is sent with the authorization request, and the verifier is sent with the token request. The authorization server recomputes the challenge from the verifier and compares it to the stored challenge, proving that the client completing the flow is the same one that started it. In OAuth 2.1, PKCE is mandatory for all authorization code clients, including confidential clients.

**Beginner-Friendly Explanation:** PKCE is like a secret handshake for OAuth. Before you start the authorization process, you create a secret (the verifier) and a corresponding proof (the challenge). You show the proof to the authorization server first, and later you reveal the secret to prove you're the same person. Even if someone steals your authorization code, they can't use it without the secret.

### Purposes

- To prevent authorization code interception attacks.
- To prevent authorization code injection attacks.
- To provide CSRF protection when combined with the state parameter.
- To protect public clients (SPAs, mobile apps) that cannot store secrets securely.

### Syntax Rules and Structure

#### PKCE Parameter Generation

```js
// S256 method (RECOMMENDED)
code_verifier = BASE64URL(RANDOM(32))
code_challenge = BASE64URL(SHA256(ASCII(code_verifier)))
```

| Parameter | Length | Characters |
|-----------|--------|------------|
| `code_verifier` | 43–128 | `[A-Z] / [a-z] / [0-9] / "-" / "." / "_" / "~"` |
| `code_challenge` | 43–128 | Same unreserved characters |

#### Method Comparison

| Method | Transformation | Security |
|--------|---------------|----------|
| S256 | `BASE64URL(SHA256(verifier))` | Recommended (MTI) |
| plain | `verifier` | Fallback only (minimal security) |

#### Syntax Rules

- The `code_verifier` MUST be generated using a cryptographically random number generator.
- The `S256` method is Mandatory To Implement (MTI) on the server.
- The `plain` method should only be used when the client cannot perform SHA-256 hashing.
- The authorization server MUST reject token requests that do not include a matching `code_verifier`.
- OAuth 2.1 makes PKCE the default requirement for all authorization code clients.

#### Constraints and Limitations

- PKCE does not protect against all token theft scenarios; sender-constrained tokens (DPoP, mTLS) provide additional protection.
- The `plain` method offers minimal security and should be avoided.

### Annotated Code Example

```js
// pkce-complete.js
const express = require('express');
const crypto = require('crypto');
const app = express();

// In-memory session store (use Redis/database in production)
const sessions = new Map();

// S256 PKCE generation
function generatePKCE() {
  const verifier = crypto.randomBytes(32).toString('base64url');
  const challenge = crypto
    .createHash('sha256')
    .update(verifier)
    .digest('base64url');
  return { verifier, challenge };
}

// Step 1: Start authorization
app.get('/auth/start', (req, res) => {
  const { verifier, challenge } = generatePKCE();
  const state = crypto.randomUUID();

  sessions.set(state, {
    verifier,
    createdAt: Date.now(),
    expiresAt: Date.now() + 600000  // 10 minutes
  });

  const authUrl = new URL('https://auth.example.com/authorize');
  authUrl.searchParams.set('response_type', 'code');
  authUrl.searchParams.set('client_id', 'spa-client');
  authUrl.searchParams.set('redirect_uri', 'http://localhost:3000/auth/callback');
  authUrl.searchParams.set('scope', 'openid profile');
  authUrl.searchParams.set('state', state);
  authUrl.searchParams.set('code_challenge', challenge);
  authUrl.searchParams.set('code_challenge_method', 'S256');

  res.redirect(authUrl.toString());
});

// Step 2: Handle callback
app.get('/auth/callback', async (req, res) => {
  const { code, state } = req.query;
  const session = sessions.get(state);

  if (!session || Date.now() > session.expiresAt) {
    return res.status(400).json({ error: 'Invalid or expired state' });
  }

  sessions.delete(state);

  // Step 3: Exchange code with PKCE verifier
  const tokenResponse = await fetch('https://auth.example.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'authorization_code',
      code,
      redirect_uri: 'http://localhost:3000/auth/callback',
      client_id: 'spa-client',
      code_verifier: session.verifier
    })
  });

  const tokens = await tokenResponse.json();
  res.json({ status: 'authenticated', tokens });
});

app.listen(3000, () => console.log('PKCE complete flow on 3000'));
```

**Expected Output (for the callback after successful authorization):**
```json
{
  "status": "authenticated",
  "tokens": {
    "access_token": "eyJhbGciOiJSUzI1NiIs...",
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_token": "def50200...",
    "scope": "openid profile"
  }
}
```

**Why this output:** The client generates a PKCE verifier and challenge, stores the verifier in a server-side session keyed by state, and redirects to the authorization server with the challenge. After the user authenticates, the authorization server redirects back with a code. The client exchanges the code for tokens, including the `code_verifier`. The authorization server recomputes the challenge from the verifier and confirms it matches the stored challenge, proving the same client initiated and completed the flow.

### Real-World Cases

- **SPAs (React, Vue, Angular):** All modern SPA auth libraries (MSAL.js, Auth0 SPA SDK) use PKCE.
- **Mobile apps (iOS, Android):** Native apps use PKCE with custom URI schemes.
- **Desktop apps (Electron):** PKCE protects the loopback redirect flow.
- **CLI tools:** PKCE secures the browser-based login flow for command-line applications.

---

## Core Concept 4: Token Management

### Definitions

**Core Definition:** Token management is the practice of issuing, storing, refreshing, and revoking access tokens (short-lived) and refresh tokens (long-lived) to maintain secure, ongoing access to protected resources.

**Technical Definition:** An **access token** is a credential used to access protected resources, typically short-lived (minutes to hours). A **refresh token** is a credential used to obtain new access tokens without requiring the user to re-authenticate, typically long-lived (hours to days). OAuth 2.1 requires that refresh tokens for public clients are either sender-constrained (e.g., DPoP) or one-time use (rotation).

**Beginner-Friendly Explanation:** Think of an access token like a day pass to a theme park — it gets you in, but it expires at the end of the day. A refresh token is like a season pass — you can use it to get a new day pass whenever you need one, without waiting in the ticket line again. Token rotation means each season pass can only be used once; using it gives you a new season pass and invalidates the old one.

### Purposes

- To provide short-lived access credentials that limit the impact of token theft.
- To enable long-lived sessions without requiring repeated user authentication.
- To detect and mitigate refresh token theft through rotation and reuse detection.
- To allow revocation of access when a user logs out or permissions change.

---

### Sub-Feature 4.1: Access Tokens vs. Refresh Tokens

#### Syntax Rules and Structure

| Token Type | Lifetime | Storage | Purpose |
|------------|----------|---------|---------|
| Access Token | Short (15 min – 1 hour) | Memory or sessionStorage | Access protected resources |
| Refresh Token | Long (hours – days) | HTTP-only cookie (browser) or secure storage (mobile) | Obtain new access tokens |

#### Syntax Rules

- Access tokens MUST be sent in the `Authorization: Bearer <token>` header.
- Bearer tokens MUST NOT be sent in query strings.
- Refresh tokens MUST be kept confidential in transit and storage.
- The authorization server MUST maintain the binding between a refresh token and the client to whom it was issued.
- Authorization servers SHOULD link refresh token lifetime to the user's authenticated session.

---

### Sub-Feature 4.2: Refresh Token Rotation

#### Definitions

**Core Definition:** Refresh token rotation is the practice of issuing a new refresh token each time the current one is used, invalidating the old one to detect and prevent token theft.

**Technical Definition:** In refresh token rotation, each refresh token can be used only once. When a client uses a refresh token to obtain new access credentials, the authorization server issues a new refresh token and invalidates the old one. If an attacker attempts to reuse a previous refresh token, the authorization server detects the reuse and invalidates the entire token family, forcing re-authentication.

**Beginner-Friendly Explanation:** Refresh token rotation is like a self-destructing key. Each time you use your key to get a new day pass, the key destroys itself and gives you a new key. If someone steals your old key and tries to use it, the system knows something is wrong and locks everything down.

#### Purposes

- To detect refresh token theft through reuse detection.
- To limit the window of opportunity for attackers with stolen refresh tokens.
- To provide defence-in-depth for public clients that cannot store secrets.

#### Syntax Rules and Structure

```js
// Server-side rotation logic
async function rotateRefreshToken(oldRefreshToken) {
  const family = await db.tokenFamilies.findByToken(oldRefreshToken);

  if (!family) {
    throw new Error('Invalid refresh token');
  }

  // Reuse detection
  if (family.currentToken !== oldRefreshToken) {
    // Token reuse detected — invalidate entire family
    await db.tokenFamilies.delete(family.familyId);
    throw new Error('Token reuse detected');
  }

  // Generate new tokens
  const newRefreshToken = generateSecureToken();
  const accessToken = generateAccessToken(family.userId);

  await db.tokenFamilies.update(family.familyId, {
    currentToken: newRefreshToken,
    lastUsed: new Date()
  });

  return { accessToken, refreshToken: newRefreshToken };
}
```

#### Syntax Rules

- Authorization servers MUST either rotate refresh tokens on each use OR use sender-constrained refresh tokens.
- Upon issuing a rotated refresh token, the authorization server MUST NOT extend the lifetime beyond the initial refresh token's expiration.
- The authorization server SHOULD set a maximum lifetime on refresh tokens OR expire them if not used within a certain period.

#### Constraints and Limitations

- Rotation requires the client to handle token updates; clients that don't may fail.
- Grace periods may be needed to account for network retries.
- Sender-constrained tokens (DPoP, mTLS) are an alternative to rotation.

#### Annotated Code Example

```js
// refresh-token-rotation.js
const express = require('express');
const crypto = require('crypto');
const app = express();
app.use(express.json());

// Simulated token family store
const tokenFamilies = new Map();

function createTokenFamily(userId) {
  const familyId = crypto.randomUUID();
  const refreshToken = crypto.randomBytes(32).toString('base64url');

  tokenFamilies.set(familyId, {
    familyId,
    userId,
    currentToken: refreshToken,
    createdAt: new Date(),
    lastUsed: new Date()
  });

  return { familyId, refreshToken };
}

app.post('/oauth/token', (req, res) => {
  const { grant_type, refresh_token } = req.body;

  if (grant_type !== 'refresh_token') {
    return res.status(400).json({ error: 'unsupported_grant_type' });
  }

  // Find the token family
  let family = null;
  for (const [id, f] of tokenFamilies) {
    if (f.currentToken === refresh_token) {
      family = f;
      break;
    }
  }

  if (!family) {
    // Check if this is a reused token
    for (const [id, f] of tokenFamilies) {
      if (f.previousToken === refresh_token) {
        // Reuse detected — invalidate entire family
        tokenFamilies.delete(id);
        return res.status(401).json({
          error: 'invalid_grant',
          error_description: 'Token reuse detected — all sessions invalidated'
        });
      }
    }
    return res.status(401).json({ error: 'invalid_grant' });
  }

  // Rotate: generate new tokens
  const newRefreshToken = crypto.randomBytes(32).toString('base64url');
  family.previousToken = family.currentToken;
  family.currentToken = newRefreshToken;
  family.lastUsed = new Date();

  res.json({
    access_token: 'at_' + crypto.randomBytes(16).toString('base64url'),
    token_type: 'Bearer',
    expires_in: 900,
    refresh_token: newRefreshToken
  });
});

app.listen(3000, () => console.log('Token rotation server on 3000'));
```

**Expected Output (for the first refresh request):**
```json
{
  "access_token": "at_abc123...",
  "token_type": "Bearer",
  "expires_in": 900,
  "refresh_token": "new_refresh_token_xyz"
}
```

**Expected Output (for reusing the old refresh token):**
```json
{
  "error": "invalid_grant",
  "error_description": "Token reuse detected — all sessions invalidated"
}
```

**Why this output:** The first refresh request uses the current refresh token, which is valid. The server rotates the tokens, issuing a new refresh token and marking the old one as `previousToken`. When an attacker (or a confused client) tries to reuse the old token, the server detects it as `previousToken` and invalidates the entire token family, forcing re-authentication.

### Real-World Cases

- **Auth0:** Refresh Token Rotation with configurable overlap period.
- **Azure Databricks:** Single-use refresh tokens enabled by default.
- **Google OAuth:** Refresh tokens for offline access; rotation recommended.

---

## Core Concept 5: Redirect URIs — Validation, Wildcard Risks, and State Parameters

### Definitions

**Core Definition:** Redirect URIs are the URLs to which the authorization server sends the user after authorization; they must be strictly validated to prevent open redirect and token leakage attacks.

**Technical Definition:** The authorization server MUST require clients to register their complete redirect URI (including the path component) and MUST reject authorization requests that specify a redirect URI that doesn't exactly match one that was registered, with an exception for loopback redirects where only the port may vary. Wildcard handling in redirect URI patterns can introduce vulnerabilities if not implemented correctly. The `state` parameter is used to maintain state between the request and callback and to prevent CSRF attacks.

**Beginner-Friendly Explanation:** A redirect URI is like a return address on an envelope. The authorization server needs to know exactly where to send the user after they approve access. If the return address is too vague (like "anywhere in this city"), an attacker could intercept the response. The `state` parameter is like a tracking number that ensures the response matches the original request.

### Purposes

- To ensure authorization responses are sent only to legitimate, pre-registered URLs.
- To prevent attackers from redirecting tokens to malicious sites.
- To protect against CSRF attacks using the state parameter.
- To prevent open redirector abuse.

### Syntax Rules and Structure

#### Exact Redirect URI Matching

```
Registered: https://app.example.com/callback
Requested:  https://app.example.com/callback  ✅ MATCH
Requested:  https://app.example.com/callback?extra=1  ❌ NO MATCH
Requested:  https://evil.example.com/callback  ❌ NO MATCH
```

#### Wildcard Risks

```
Registered: https://*.example.com/callback
Risk: An attacker who controls a subdomain (e.g., via subdomain takeover)
can receive authorization codes.
```

#### State Parameter

```js
// Generate state (CSRF token)
const state = crypto.randomUUID();
sessionStorage.setItem('oauth_state', state);

// Include in authorization request
authUrl.searchParams.set('state', state);

// Verify on callback
const returnedState = new URLSearchParams(location.search).get('state');
if (returnedState !== sessionStorage.getItem('oauth_state')) {
  throw new Error('CSRF detected');
}
```

#### Syntax Rules

- The authorization server MUST use simple string comparison for redirect URI matching.
- Wildcard patterns in redirect URIs can lead to subdomain takeover attacks.
- The client MUST NOT expose URLs that forward the browser to arbitrary URIs obtained from a query parameter ("open redirector").
- The `state` parameter MUST be used for CSRF protection unless PKCE is relied upon.

#### Constraints and Limitations

- Exact matching does not work for native apps using loopback redirects; port may vary.
- Wildcards are inherently risky and should be avoided.
- The `state` parameter must be bound to the user agent session.

### Annotated Code Example

```js
// redirect-uri-validation.js
const express = require('express');
const crypto = require('crypto');
const app = express();

// Registered redirect URIs (from client registration)
const registeredRedirectUris = [
  'https://app.example.com/callback',
  'https://app.example.com/auth/callback'
];

// Validate redirect URI using exact match
function isValidRedirectUri(uri) {
  return registeredRedirectUris.includes(uri);
}

// Authorization endpoint
app.get('/authorize', (req, res) => {
  const { client_id, redirect_uri, state, code_challenge } = req.query;

  // Validate redirect URI
  if (!isValidRedirectUri(redirect_uri)) {
    return res.status(400).json({
      error: 'invalid_request',
      error_description: 'Redirect URI not registered'
    });
  }

  // Generate authorization code
  const code = crypto.randomBytes(32).toString('base64url');

  // Build redirect URL with code and state
  const redirectUrl = new URL(redirect_uri);
  redirectUrl.searchParams.set('code', code);
  if (state) redirectUrl.searchParams.set('state', state);

  res.redirect(redirectUrl.toString());
});

// Callback endpoint (client-side)
app.get('/callback', (req, res) => {
  const { code, state } = req.query;

  // Verify state parameter
  const expectedState = req.session?.oauthState;
  if (!expectedState || state !== expectedState) {
    return res.status(400).json({ error: 'Invalid state parameter — CSRF detected' });
  }

  res.json({
    message: 'Authorization successful',
    code,
    state_verified: true
  });
});

app.listen(3000, () => console.log('Redirect URI validation on 3000'));
```

**Expected Output (for a valid redirect URI):**
```
HTTP/1.1 302 Found
Location: https://app.example.com/callback?code=abc123...&state=xyz789...
```

**Expected Output (for an invalid redirect URI):**
```json
{
  "error": "invalid_request",
  "error_description": "Redirect URI not registered"
}
```

**Why this output:** The authorization server compares the requested `redirect_uri` against the registered URIs using exact string matching. If it matches, the user is redirected with the authorization code and state. If not, the request is rejected with an error, preventing token leakage to unauthorised URLs.

### Real-World Cases

- **Google OAuth:** Requires exact redirect URI matching; wildcards not supported.
- **Okta:** Sign-in redirect URIs must be exact, case-sensitive matches including trailing slashes.
- **Microsoft Entra ID:** Redirect URIs for SPAs must be configured with the `spa` type and exact matches.
- **CVE-2026-7504 (Keycloak):** Wildcard redirect URI vulnerability allowed attackers to redirect to unauthorised URLs.

---

## References

- RFC 9700 — Best Current Practice for OAuth 2.0 Security (BCP 240) — https://www.rfc-editor.org/rfc/rfc9700
- RFC 7636 — Proof Key for Code Exchange (PKCE) — https://www.rfc-editor.org/rfc/rfc7636
- RFC 6749 — The OAuth 2.0 Authorization Framework — https://www.rfc-editor.org/rfc/rfc6749
- OAuth 2.1 Authorization Framework (IETF Draft) — https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/
- OAuth 2.0 for Browser-Based Applications (IETF Draft) — https://datatracker.ietf.org/doc/draft-ietf-oauth-browser-based-apps/
- RFC 8252 — OAuth 2.0 for Native Apps — https://www.rfc-editor.org/rfc/rfc8252
- RFC 9449 — OAuth 2.0 Demonstrating Proof of Possession (DPoP) — https://www.rfc-editor.org/rfc/rfc9449
- OAuth 2.0 Security Best Current Practice — https://datatracker.ietf.org/doc/rfc9700/
- OAuth 2.1 Changes Summary — https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-15
- PKCE Documentation — https://oauth.net/2/pkce/
- Auth0 Refresh Token Rotation — https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation
- Microsoft identity platform OAuth 2.0 authorization code flow — https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow
- OWASP OAuth 2.0 Security Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/OAuth2_Cheat_Sheet.html