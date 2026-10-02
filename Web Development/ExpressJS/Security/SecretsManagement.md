# Express.js Secrets Management — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Secrets management is the practice of securely storing, accessing, rotating, and auditing sensitive credentials — such as API keys, database passwords, JWT signing keys, and encryption keys — throughout their lifecycle, ensuring they never appear in source code, version control, or logs.

**Technical Definition:** Secrets management encompasses the tooling, processes, and runtime patterns for handling credentials that grant access to systems and data. The OWASP Secrets Management Cheat Sheet defines the full lifecycle as creation, storage, access control, rotation, revocation, expiration, and auditing, and recommends centralised management because secrets often spread beyond the systems meant to control them. In Node.js, secrets are typically injected via environment variables, fetched from managed secret stores (AWS Secrets Manager, HashiCorp Vault, Google Cloud Secret Manager), or generated as short-lived credentials via OIDC.

**Beginner-Friendly Explanation:** Think of secrets as the keys to your house, car, and office. Secrets management is the discipline of keeping those keys in a locked safe (not under the doormat), giving each person only the key they need, changing the locks regularly, and keeping a log of who opened the safe. In software, the "keys" are passwords and API tokens, and the "safe" is a secrets manager.

### Key Characteristics

- **Lifecycle-oriented:** Secrets have a creation, active, rotation, and revocation phase; each must be managed.
- **Defence-in-depth:** No single control is sufficient; environment variables, secret stores, scanning tools, and rotation must be layered.
- **Zero-standing-credential ideal:** The goal is to eliminate long-lived secrets where possible, replacing them with short-lived, dynamically issued credentials.
- **Fail-fast validation:** Missing or malformed secrets should crash the application at startup, not cause silent failures at runtime.
- **Audit-required:** Every secret access must be logged and traceable to a workload, owner, and timestamp.

### Prerequisites

- **Node.js runtime** (v18 or higher; Node 20.6+ supports native `.env` loading via `--env-file`).
- **Express.js installed** (`npm install express`).
- **Basic understanding of environment variables and `process.env`**.
- **Familiarity with cloud IAM concepts** (roles, policies, workload identity).
- **Access to a secret store** (AWS Secrets Manager, Vault, or Google Secret Manager) for production use.

### Related Programming Areas

- **Cloud infrastructure:** IAM roles, managed identities, and OIDC-based authentication.
- **Cryptography:** Key generation, signing algorithms (RS256, ES256), and key rotation.
- **CI/CD security:** Pipeline secret injection, scanning, and environment separation.
- **Database administration:** Least-privilege roles, credential rotation, and connection pooling.
- **Application security:** Preventing injection, XSS, and information disclosure through secret leakage.

### Core Concepts

1. **Environment Variables** — using `.env` safely, structuring fallback defaults, and preventing runtime modification leaks.
2. **Secret Managers** — integrating AWS Secrets Manager, HashiCorp Vault, and Google Cloud Secret Manager.
3. **API Keys & Token Signatures** — securing asymmetry, short-lived JWT lifetimes, revocation, and signing algorithms.
4. **Database Credentials** — least-privilege access and separate migration vs. runtime roles.
5. **Rotating Secrets** — seamless cryptographic key rotation and database password updates without downtime.
6. **Never Committing Secrets** — git-secrets, gitleaks, pre-commit hooks, and CI/CD secret scanning.

---

## Core Concept 1: Environment Variables

### Definitions

**Core Definition:** Environment variables are key-value pairs injected into a process's environment at runtime, providing a way to pass configuration and secrets to an application without hardcoding them in source files.

**Technical Definition:** In Node.js, environment variables are accessible via `process.env`. The `dotenv` package loads key-value pairs from a `.env` file into `process.env` at application startup. Node.js 20.6+ supports native `.env` file loading via the `--env-file` CLI flag, eliminating the `dotenv` dependency for basic use cases. The `dotenv` library itself has no known vulnerabilities; the security risk lies entirely in how the `.env` file is handled around it.

**Beginner-Friendly Explanation:** Environment variables are like the settings on a thermostat — they tell the application how to behave without being part of the application's code. A `.env` file is a notepad where you write down those settings for local development. The danger is leaving that notepad on the kitchen table where anyone can read it.

### Purposes

- To separate configuration and secrets from source code, enabling the same code to run in different environments.
- To provide a standard interface (`process.env`) for reading configuration across platforms and deployment targets.
- To enable local development with `.env` files while using managed secret stores in production.
- To validate required variables at startup, failing fast rather than silently using insecure defaults.

### Sub-Feature 1.1: Using `.env` Safely

#### Definitions

**Core Definition:** Safe `.env` usage means keeping the `.env` file out of version control, out of Docker build contexts, and out of logs, while committing only a `.env.example` template with placeholder values.

**Technical Definition:** The `.env` file must be added to `.gitignore` and `.dockerignore` before the first commit. A `.env.example` file containing all required keys but no real values should be committed to document the expected configuration. Environment variables must never be logged or included in crash dumps.

**Beginner-Friendly Explanation:** Your `.env` file is like a personal diary of passwords. You keep it in a locked drawer (`.gitignore`), never photocopy it into a shared folder (version control), and leave a blank template on the fridge (`.env.example`) so family members know what fields to fill in.

#### Purposes

- To prevent production credentials from entering version control history.
- To provide a safe, documented template for onboarding new developers.
- To ensure that Docker images do not contain baked-in secrets.

#### Syntax Rules and Structure

```bash
# .gitignore
.env
.env.*
!.env.example

# .dockerignore
.env
.env.*
```

```js
// config.js — Load and validate environment variables
require('dotenv').config();

const required = ['DATABASE_URL', 'JWT_SECRET', 'API_KEY'];
const missing = required.filter((key) => !process.env[key]);
if (missing.length > 0) {
  throw new Error(`Missing env vars: ${missing.join(', ')}`);
}
```

| Component | Breakdown |
|-----------|-----------|
| `.gitignore` | Prevents `.env` from being committed. |
| `!.env.example` | Exception: allows the example file to be committed. |
| `dotenv.config()` | Loads `.env` into `process.env`. |
| Validation | Throws at startup if required variables are missing. |

**Constraints and Limitations:**
- `.env` files are plain text and readable by anything in the process; they are not suitable for production secrets.
- Deleting a committed `.env` file does not remove it from git history; secrets must be rotated and history scrubbed.
- Environment variables appear in crash dumps and can be accidentally logged.

#### Annotated Code Example

```js
// env-safe.js — Safe .env usage with validation
require('dotenv').config();

// ✅ Validate required variables at startup
const required = ['DATABASE_URL', 'JWT_SECRET', 'API_KEY'];
const missing = required.filter((key) => !process.env[key]);

if (missing.length > 0) {
  console.error(`[FATAL] Missing environment variables: ${missing.join(', ')}`);
  process.exit(1); // Fail fast
}

// ✅ Never log raw environment
console.log('Environment loaded:', {
  NODE_ENV: process.env.NODE_ENV,
  DATABASE_URL: process.env.DATABASE_URL ? '[SET]' : '[MISSING]',
  JWT_SECRET: process.env.JWT_SECRET ? '[SET]' : '[MISSING]'
});

const express = require('express');
const app = express();

app.get('/', (req, res) => res.json({ status: 'ok' }));
app.listen(3000, () => console.log('Server on 3000'));
```

**Expected Output (server console with all variables set):**
```
Environment loaded: { NODE_ENV: 'development', DATABASE_URL: '[SET]', JWT_SECRET: '[SET]' }
Server on 3000
```

**Expected Output (missing variables):**
```
[FATAL] Missing environment variables: DATABASE_URL, JWT_SECRET
(process exits with code 1)
```

**Why this output:** The validation step checks for required variables before the server starts. If any are missing, the process exits immediately with a clear message. The logging example shows how to indicate whether a secret is set without printing the actual value.

#### Real-World Cases

- **Local development:** Developers copy `.env.example` to `.env` and fill in development credentials.
- **Staging and production:** Secrets are injected via the platform (Heroku config vars, AWS ECS task definitions) rather than `.env` files.
- **Docker:** `.dockerignore` excludes `.env` from the build context; secrets are mounted at runtime.

---

### Sub-Feature 1.2: Structuring Fallback Defaults

#### Definitions

**Core Definition:** Fallback defaults are safe, non-sensitive values used when an environment variable is not set, allowing the application to run in development without requiring every secret to be configured.

**Technical Definition:** The `||` operator or `??` nullish coalescing operator provides fallback values. Sensitive secrets should never have fallback defaults; only non-critical configuration (port numbers, log levels, feature flags) should. Libraries like `envalid` provide type coercion and validation with clearer error messages.

**Beginner-Friendly Explanation:** A fallback default is like a "Plan B" — if the specific setting isn't provided, the application uses a safe, sensible default. But you never fall back on a password; if a password is missing, the application should stop.

#### Purposes

- To allow the application to start in development without every secret configured.
- To provide sensible defaults for non-sensitive configuration (ports, timeouts).
- To fail fast when critical secrets are missing rather than silently using insecure defaults.

#### Syntax Rules and Structure

```js
// ✅ Safe fallback for non-sensitive config
const PORT = process.env.PORT || 3000;
const LOG_LEVEL = process.env.LOG_LEVEL || 'info';

// ❌ NEVER use a fallback for secrets
// const JWT_SECRET = process.env.JWT_SECRET || 'dev-secret'; // DANGEROUS

// ✅ Fail fast for secrets
const JWT_SECRET = process.env.JWT_SECRET;
if (!JWT_SECRET) throw new Error('JWT_SECRET is required');
```

| Component | Breakdown |
|-----------|-----------|
| `||` / `??` | Provides fallback for non-sensitive values. |
| Missing secret | Throws an error at startup. |
| `envalid` | Type-safe validation with defaults for non-secrets. |

**Constraints and Limitations:**
- A fallback secret (e.g., `'dev-secret'`) can accidentally be used in production if the environment variable is not set.
- Fallbacks must be clearly documented as development-only.

#### Annotated Code Example

```js
// env-fallbacks.js — Safe fallbacks for non-sensitive config
require('dotenv').config();

const express = require('express');
const app = express();

// ✅ Safe fallbacks for non-sensitive configuration
const PORT = process.env.PORT || 3000;
const LOG_LEVEL = process.env.LOG_LEVEL || 'info';
const NODE_ENV = process.env.NODE_ENV || 'development';

// ✅ Required secrets — no fallback
const JWT_SECRET = process.env.JWT_SECRET;
if (!JWT_SECRET) {
  throw new Error('JWT_SECRET must be set. Generate one with: openssl rand -hex 32');
}

app.get('/config', (req, res) => {
  res.json({ PORT, LOG_LEVEL, NODE_ENV, JWT_SECRET_SET: !!JWT_SECRET });
});

app.listen(PORT, () => console.log(`Server on ${PORT} (${NODE_ENV})`));
```

**Expected Output (with `JWT_SECRET` set, `PORT` unset):**
```
Server on 3000 (development)
```

**Expected Output (with `JWT_SECRET` unset):**
```
Error: JWT_SECRET must be set. Generate one with: openssl rand -hex 32
```

**Why this output:** The `PORT` falls back to `3000` because it is non-sensitive. `JWT_SECRET` has no fallback; if it is missing, the application throws an error immediately, preventing insecure operation.

#### Real-World Cases

- **Development:** `PORT` falls back to 3000, `LOG_LEVEL` falls back to `'debug'`.
- **Staging:** Environment variables are set explicitly; fallbacks are never used.
- **Production:** All variables including secrets are set explicitly; missing secrets cause startup failure.

---

### Sub-Feature 1.3: Preventing Runtime Modification Leaks

#### Definitions

**Core Definition:** Runtime modification leaks occur when an attacker or malicious dependency reads, modifies, or logs environment variables after the application has started.

**Technical Definition:** `process.env` is mutable at runtime. A compromised dependency or an XSS payload executing in a child process could read `process.env` and exfiltrate secrets. Node.js 20+ supports the `--permission` model, which restricts filesystem access and prevents dependencies from reading files they were not granted. Additionally, secrets should be loaded into a frozen configuration object at startup and not read from `process.env` directly throughout the codebase.

**Beginner-Friendly Explanation:** Once the application is running, `process.env` is like an open book that any code in the process can read. A malicious package could read the book and send your passwords to an attacker. The solution is to copy the important pages into a locked box at startup and then close the book.

#### Purposes

- To limit the exposure of secrets to only the code that needs them.
- To prevent malicious dependencies from reading environment variables they were not granted access to.
- To provide a single, auditable point of secret access.

#### Syntax Rules and Structure

```js
// config.js — Frozen configuration object
const config = Object.freeze({
  port: parseInt(process.env.PORT || '3000', 10),
  dbUrl: process.env.DATABASE_URL,
  jwtSecret: process.env.JWT_SECRET,
  apiKey: process.env.API_KEY
});

// Other modules import config instead of reading process.env directly
module.exports = config;
```

| Component | Breakdown |
|-----------|-----------|
| `Object.freeze()` | Prevents runtime modification of the config object. |
| Single module | Centralises secret access for auditing. |
| `--permission` | Node.js flag restricting filesystem and child process access. |

**Constraints and Limitations:**
- `Object.freeze()` is shallow; nested objects must be frozen individually.
- The `--permission` model is experimental in Node.js and may not be suitable for all applications.
- A compromised dependency running in the same process can still read `process.env` unless the permission model is used.

#### Annotated Code Example

```js
// config.js — Frozen configuration object
require('dotenv').config();

const config = Object.freeze({
  port: parseInt(process.env.PORT || '3000', 10),
  nodeEnv: process.env.NODE_ENV || 'development',
  db: Object.freeze({
    url: process.env.DATABASE_URL,
    poolSize: parseInt(process.env.DB_POOL_SIZE || '10', 10)
  }),
  jwt: Object.freeze({
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '1h'
  })
});

// Validate required secrets
if (!config.db.url) throw new Error('DATABASE_URL is required');
if (!config.jwt.secret) throw new Error('JWT_SECRET is required');

module.exports = config;
```

**Expected Output (when imported by another module):**
```
(config object is frozen; attempts to modify config.jwt.secret throw in strict mode)
```

**Why this output:** `Object.freeze()` prevents modification of the top-level properties. The nested `db` and `jwt` objects are also frozen. Other modules import `config` and use `config.db.url` instead of `process.env.DATABASE_URL`, providing a single point of access and audit.

#### Real-World Cases

- **Microservices:** Each service loads its own frozen config at startup; shared configuration is validated once.
- **Monorepos:** A shared `config` package centralises secret loading for all workspaces.
- **Serverless functions:** Cold-start initialisation loads and freezes config before the first request.

---

## Core Concept 2: Secret Managers

### Definitions

**Core Definition:** A secret manager is a centralised service designed to store, access, rotate, and audit secrets, providing encryption at rest, access control, and audit trails that flat files cannot.

**Technical Definition:** AWS Secrets Manager, HashiCorp Vault, and Google Cloud Secret Manager are managed services that store secrets encrypted at rest and provide SDKs for programmatic retrieval. Applications authenticate to the secret manager using platform identity (IAM role, workload identity, AppRole), eliminating the need for a bootstrap secret. Secrets are fetched at startup and cached in memory for the application's lifetime.

**Beginner-Friendly Explanation:** A secret manager is like a bank vault for passwords. Instead of keeping passwords in a notebook, you store them in the vault. When your application needs a password, it shows its ID badge (IAM role) to the vault, and the vault hands over the password. The vault records every access, so you know who took what and when.

### Purposes

- To provide encrypted storage for secrets at rest and in transit.
- To enable fine-grained access control based on workload identity.
- To provide audit trails of every secret access.
- To support centralised rotation without application redeployment.
- To eliminate hardcoded credentials and bootstrap secrets.

### Sub-Feature 2.1: AWS Secrets Manager Integration

#### Definitions

**Core Definition:** AWS Secrets Manager is a managed service that stores and rotates secrets, accessible from Node.js via the `@aws-sdk/client-secrets-manager` package.

**Technical Definition:** The `SecretsManagerClient` is configured with a region and uses the SDK's credential provider chain (IAM role, environment variables, shared credentials file) for authentication. The `GetSecretValueCommand` retrieves a secret by ID, and the response's `SecretString` property contains the secret value as a string (typically JSON). The SDK caches no values; the application should fetch secrets at startup and store them in memory.

**Beginner-Friendly Explanation:** AWS Secrets Manager is Amazon's password vault. Your application (running on EC2, ECS, or Lambda) has an IAM role that says "you can read the database password from the vault." The application calls the vault, gets the password, and uses it to connect to the database.

#### Purposes

- To eliminate hardcoded database credentials from source code and configuration files.
- To leverage AWS IAM for access control and audit logging.
- To support automatic rotation of RDS and Aurora credentials.

#### Syntax Rules and Structure

```js
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

const client = new SecretsManagerClient({ region: 'us-east-1' });

async function getSecret(secretId) {
  const response = await client.send(new GetSecretValueCommand({ SecretId: secretId }));
  return JSON.parse(response.SecretString);
}
```

| Component | Breakdown |
|-----------|-----------|
| `SecretsManagerClient` | Configured with region; uses IAM credentials. |
| `GetSecretValueCommand` | Retrieves the secret by ID. |
| `SecretString` | The secret value as a string (often JSON). |

**Constraints and Limitations:**
- Requires an IAM role or credentials with `secretsmanager:GetSecretValue` permission.
- Secrets Manager has API rate limits; fetch at startup and cache in memory.
- The SDK credential provider chain must be able to resolve credentials; explicit keys should not be hardcoded.

#### Annotated Code Example

```js
// aws-secrets.js — AWS Secrets Manager integration
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';
import express from 'express';

const client = new SecretsManagerClient({ region: process.env.AWS_REGION || 'us-east-1' });

async function loadSecret(secretId) {
  const response = await client.send(new GetSecretValueCommand({ SecretId: secretId }));
  return JSON.parse(response.SecretString);
}

(async () => {
  // ✅ Fetch secrets at startup
  const dbCredentials = await loadSecret('prod/database');
  const apiKeys = await loadSecret('prod/api-keys');

  const app = express();

  app.get('/health', (req, res) => {
    res.json({
      db: { host: dbCredentials.host, user: dbCredentials.username, password: '[REDACTED]' },
      apiKeys: Object.keys(apiKeys)
    });
  });

  app.listen(3000, () => console.log('AWS Secrets Manager server on 3000'));
})();
```

**Expected Output (server console):**
```
AWS Secrets Manager server on 3000
```

**Expected Output (for `GET /health`):**
```
{
  "db": { "host": "prod-db.cluster-abc.us-east-1.rds.amazonaws.com", "user": "app_user", "password": "[REDACTED]" },
  "apiKeys": ["stripe", "sendgrid", "openai"]
}
```

**Why this output:** The secrets are fetched at startup and stored in the closure variables `dbCredentials` and `apiKeys`. The health endpoint exposes only non-sensitive metadata (host, username, key names) while redacting the actual password values.

#### Real-World Cases

- **ECS Fargate microservices:** The task execution role has permission to read secrets from Secrets Manager; the application fetches them at startup.
- **Lambda functions:** The execution role grants access; secrets are fetched during cold start and cached in the execution environment.
- **EC2 instances:** The instance profile role grants access; secrets are fetched at boot.

---

### Sub-Feature 2.2: HashiCorp Vault Integration

#### Definitions

**Core Definition:** HashiCorp Vault is an open-source secret management tool that provides a unified interface for secrets, encryption keys, and dynamic credentials, accessible from Node.js via third-party clients like `node-vault` or `nanvc`.

**Technical Definition:** Vault uses authentication methods (AppRole, Kubernetes, OIDC, token) to grant access to secret engines (KV v1/v2, database, PKI). The `nanvc` TypeScript client provides typed KV operations, AppRole login, and database credential generation. The `node-vault` package is a widely used alternative.

**Beginner-Friendly Explanation:** Vault is like a Swiss Army knife for secrets. It can store static passwords (KV engine), generate dynamic database credentials on demand (database engine), and issue TLS certificates (PKI engine). Your application authenticates to Vault using a role ID and secret ID, then asks for what it needs.

#### Purposes

- To support dynamic, short-lived credentials that expire automatically.
- To provide a unified interface for multiple secret types (static, dynamic, encryption keys).
- To enable fine-grained access control through Vault policies.
- To integrate with Kubernetes, cloud IAM, and CI/CD systems.

#### Syntax Rules and Structure

```js
import { VaultClientV2 } from 'nanvc';

const vault = new VaultClientV2();

// KV v2 read
const secret = await vault.read('secret-v2', 'apps/demo', { engineVersion: 2 }).unwrap();

// AppRole login
const login = await vault.auth.loginWithAppRole({
  role_id: process.env.VAULT_ROLE_ID,
  secret_id: process.env.VAULT_SECRET_ID
}).unwrap();
```

| Component | Breakdown |
|-----------|-----------|
| `VaultClientV2` | Typed client for Vault KV v2 and other engines. |
| `read()` | Reads a secret from the KV engine. |
| `loginWithAppRole()` | Authenticates using role ID and secret ID. |
| `unwrap()` | Extracts the value from the Result type. |

**Constraints and Limitations:**
- Vault must be deployed and managed separately (or used as a managed service).
- AppRole secret IDs are themselves secrets; they must be delivered to the application securely (e.g., via response wrapping).
- The `node-vault` package is community-maintained and not official HashiCorp SDK.

#### Annotated Code Example

```js
// vault-secrets.js — HashiCorp Vault with nanvc
import { VaultClientV2 } from 'nanvc';
import express from 'express';

const vault = new VaultClientV2({ endpoint: process.env.VAULT_ADDR || 'http://localhost:8200' });

async function loadSecrets() {
  // ✅ AppRole authentication
  await vault.auth.loginWithAppRole({
    role_id: process.env.VAULT_ROLE_ID,
    secret_id: process.env.VAULT_SECRET_ID
  }).unwrap();

  // ✅ Read KV v2 secrets
  const db = await vault.read('secret-v2', 'apps/db', { engineVersion: 2 }).unwrap();
  const api = await vault.read('secret-v2', 'apps/api', { engineVersion: 2 }).unwrap();

  return { db, api };
}

(async () => {
  const { db, api } = await loadSecrets();

  const app = express();

  app.get('/status', (req, res) => {
    res.json({
      db: { host: db.host, username: db.username, password: '[REDACTED]' },
      api: Object.keys(api)
    });
  });

  app.listen(3000, () => console.log('Vault server on 3000'));
})();
```

**Expected Output (server console):**
```
Vault server on 3000
```

**Expected Output (for `GET /status`):**
```
{
  "db": { "host": "db.internal", "username": "app_user", "password": "[REDACTED]" },
  "api": ["stripe_key", "sendgrid_key"]
}
```

**Why this output:** Vault authenticates via AppRole, reads the KV v2 secrets, and returns only non-sensitive metadata to the client. The actual secret values are kept in memory and never exposed.

#### Real-World Cases

- **Kubernetes:** Vault integrates with Kubernetes service accounts for pod authentication.
- **Multi-cloud:** Vault provides a consistent secret management interface across AWS, GCP, and Azure.
- **Dynamic database credentials:** Vault generates unique, short-lived database users for each application instance.

---

### Sub-Feature 2.3: Google Cloud Secret Manager Integration

#### Definitions

**Core Definition:** Google Cloud Secret Manager is a managed service for storing and accessing secrets, accessible from Node.js via the `@google-cloud/secret-manager` package.

**Technical Definition:** The `SecretManagerServiceClient` is used to access secrets. Authentication uses Application Default Credentials (ADC), which resolve to a service account attached to the compute resource (Cloud Run, GKE, GCE). The `accessSecretVersion` method retrieves the secret payload as a Buffer.

**Beginner-Friendly Explanation:** Google Cloud Secret Manager is Google's password vault. Your application (running on Cloud Run or GKE) uses a service account to prove its identity and retrieve secrets from the vault.

#### Purposes

- To store and retrieve secrets in Google Cloud environments without hardcoded credentials.
- To leverage Google Cloud IAM for access control and audit logging.
- To support automatic replication and rotation policies.

#### Syntax Rules and Structure

```js
const { SecretManagerServiceClient } = require('@google-cloud/secret-manager');
const client = new SecretManagerServiceClient();

async function accessSecret(projectId, secretId) {
  const name = `projects/${projectId}/secrets/${secretId}/versions/latest`;
  const [version] = await client.accessSecretVersion({ name });
  return version.payload.data.toString('utf8');
}
```

| Component | Breakdown |
|-----------|-----------|
| `SecretManagerServiceClient` | Client for the Secret Manager API. |
| `accessSecretVersion` | Retrieves a specific secret version. |
| `payload.data` | The secret value as a Buffer. |

**Constraints and Limitations:**
- Requires the Secret Manager API to be enabled and the service account to have the `secretmanager.versions.access` permission.
- Uses Application Default Credentials (ADC); no explicit key files should be used in production.
- Secret versions are immutable; updating a secret creates a new version.

#### Annotated Code Example

```js
// gcp-secrets.js — Google Cloud Secret Manager
const { SecretManagerServiceClient } = require('@google-cloud/secret-manager');
const express = require('express');

const client = new SecretManagerServiceClient();
const projectId = process.env.GCP_PROJECT_ID;

async function getSecret(secretId) {
  const name = `projects/${projectId}/secrets/${secretId}/versions/latest`;
  const [version] = await client.accessSecretVersion({ name });
  return version.payload.data.toString('utf8');
}

(async () => {
  const dbUrl = await getSecret('database-url');
  const jwtSecret = await getSecret('jwt-secret');

  const app = express();

  app.get('/status', (req, res) => {
    res.json({
      dbUrlSet: !!dbUrl,
      jwtSecretSet: !!jwtSecret,
      dbUrl: '[REDACTED]'
    });
  });

  app.listen(3000, () => console.log('GCP Secret Manager server on 3000'));
})();
```

**Expected Output (for `GET /status`):**
```
{"dbUrlSet":true,"jwtSecretSet":true,"dbUrl":"[REDACTED]"}
```

**Why this output:** The secrets are fetched at startup and stored in closure variables. The status endpoint reports whether each secret was successfully retrieved without exposing the values.

#### Real-World Cases

- **Cloud Run services:** The service account attached to the Cloud Run revision has permission to access secrets.
- **GKE workloads:** Workload Identity binds a Kubernetes service account to a Google service account with secret access.
- **Cloud Functions:** The function's service account grants access to Secret Manager.

---

## Core Concept 3: API Keys & Token Signatures

### Definitions

**Core Definition:** API keys and token signatures are cryptographic mechanisms for authenticating clients and verifying the integrity of tokens, with security depending on key strength, algorithm choice, and token lifetime management.

**Technical Definition:** JSON Web Tokens (JWTs) are signed using either symmetric algorithms (HS256, shared secret) or asymmetric algorithms (RS256, ES256, EdDSA, private/public key pair). Asymmetric algorithms are preferred in distributed systems because the signing key is held by a single issuer while public keys can be distributed for verification. Short token lifetimes limit the window of opportunity for stolen tokens. Token revocation requires a blacklist or denylist keyed by the JWT ID (`jti`) claim.

**Beginner-Friendly Explanation:** A JWT is like a signed concert ticket. The signature proves the ticket is genuine. Symmetric signing is like a secret handshake — both the ticket seller and the gatekeeper need to know the handshake. Asymmetric signing is like a wax seal — only the seller has the stamp, but anyone can verify the seal's shape. Short expiration is like a ticket that's only valid for one night.

### Purposes

- To authenticate API clients and users without transmitting passwords on every request.
- To ensure token integrity through cryptographic signatures.
- To limit the impact of token theft through short lifetimes and revocation.
- To support distributed verification across multiple services without sharing signing keys.

### Sub-Feature 3.1: Choosing Secure Signing Algorithms

#### Definitions

**Core Definition:** The signing algorithm determines the cryptographic mechanism used to sign a JWT, with asymmetric algorithms (RS256, ES256, EdDSA) preferred over symmetric (HS256) in distributed systems.

**Technical Definition:** HMAC (HS256) uses a shared secret for both signing and verification. RSA (RS256) uses a private key for signing and a public key for verification. ECDSA (ES256) uses elliptic curve keys with smaller key sizes and faster operations. EdDSA (Ed25519) is the newest and most secure option. `alg: 'none'` must be explicitly rejected to prevent algorithm confusion attacks.

**Beginner-Friendly Explanation:** Think of signing algorithms as different types of locks. HS256 is a padlock where everyone who can open it also has the key to lock it — fine if you trust everyone. RS256/ES256/EdDSA are padlocks where only one person can lock it, but anyone can check that it's locked.

#### Purposes

- To prevent algorithm confusion attacks (e.g., `alg: 'none'` or RS256 downgraded to HS256).
- To enable distributed verification where multiple services verify tokens without possessing the signing key.
- To provide stronger security with smaller key sizes (EC vs. RSA).

#### Syntax Rules and Structure

```js
import { SignJWT, jwtVerify } from 'jose';

// ✅ Asymmetric signing (ES256)
const jwt = await new SignJWT({ userId: '123' })
  .setProtectedHeader({ alg: 'ES256' })
  .setIssuedAt()
  .setExpirationTime('1h')
  .setJti(crypto.randomUUID())
  .sign(privateKey);

// ✅ Verify with explicit algorithm allowlist
const { payload } = await jwtVerify(token, publicKey, {
  algorithms: ['ES256', 'RS256', 'EdDSA'], // Reject HS256 and none
  issuer: 'myapp',
  audience: 'myapi'
});
```

| Algorithm | Key Type | Use Case |
|-----------|----------|----------|
| HS256 | Shared secret | Single service, simple setups |
| RS256 | RSA key pair | Legacy systems, wide support |
| ES256 | EC key pair | New deployments, smaller keys |
| EdDSA | Ed25519 key pair | Newest, strongest option |

**Constraints and Limitations:**
- `alg: 'none'` must be explicitly rejected; some libraries default to allowing it.
- Algorithm allowlists must be enforced on verification.
- Key rotation must be supported via a JWKS endpoint or key ID (`kid`) header.

#### Annotated Code Example

```js
// jwt-signing.js — Asymmetric JWT with ES256
const { SignJWT, jwtVerify, generateKeyPair } = require('jose');
const express = require('express');
const crypto = require('node:crypto');

(async () => {
  // Generate ES256 key pair (in production, load from secret manager)
  const { publicKey, privateKey } = await generateKeyPair('ES256');

  const app = express();
  app.use(express.json());

  // Issue token
  app.post('/login', async (req, res) => {
    const jwt = await new SignJWT({ userId: req.body.userId })
      .setProtectedHeader({ alg: 'ES256' })
      .setIssuedAt()
      .setExpirationTime('15m')       // ✅ Short-lived
      .setJti(crypto.randomUUID())
      .sign(privateKey);

    res.json({ token: jwt });
  });

  // Verify token
  app.get('/protected', async (req, res) => {
    const token = req.headers.authorization?.replace('Bearer ', '');
    try {
      const { payload } = await jwtVerify(token, publicKey, {
        algorithms: ['ES256'],        // ✅ Explicit algorithm allowlist
        issuer: 'myapp'
      });
      res.json({ userId: payload.userId, jti: payload.jti });
    } catch (err) {
      res.status(401).json({ error: 'Invalid token' });
    }
  });

  app.listen(3000, () => console.log('ES256 JWT server on 3000'));
})();
```

**Expected Output (for `POST /login` with `{"userId":"alice"}`):**
```
{"token":"eyJhbGciOiJFUzI1NiIs..."}
```

**Expected Output (for `GET /protected` with a valid token):**
```
{"userId":"alice","jti":"a1b2c3d4-..."}
```

**Why this output:** The token is signed with ES256 using the private key. Verification uses only the public key and an explicit algorithm allowlist that rejects HS256 and none. The `jti` claim provides a unique identifier for revocation.

#### Real-World Cases

- **OAuth 2.0 / OpenID Connect:** Asymmetric algorithms enable resource servers to verify tokens using the authorization server's JWKS endpoint.
- **Microservices:** Each service verifies tokens locally using the public key, without calling a central service.
- **API gateways:** The gateway verifies tokens and forwards claims to backend services.

---

### Sub-Feature 3.2: Short-Lived Tokens and Revocation

#### Definitions

**Core Definition:** Short-lived tokens expire quickly, limiting the window of opportunity for stolen tokens. Revocation uses a blacklist (denylist) to invalidate tokens before their natural expiration.

**Technical Definition:** Access tokens should have lifetimes of 15 minutes or less. Refresh tokens have longer lifetimes but must be revocable. Revocation is implemented by storing the `jti` claim in a Redis-backed blacklist with a TTL matching the token's remaining lifetime. The `jwt-plug` package provides in-memory and Redis-backed blacklist adapters.

**Beginner-Friendly Explanation:** A short-lived token is like a movie ticket that's only valid for one showing. If someone steals it, they can only use it for that one showing. Revocation is like the theater keeping a list of stolen tickets and refusing entry to anyone presenting them.

#### Purposes

- To limit the damage from stolen or leaked tokens.
- To support logout by invalidating tokens before expiration.
- To enable immediate revocation of compromised tokens.
- To balance security (short lifetime) with user experience (refresh tokens).

#### Syntax Rules and Structure

```js
// Redis-backed blacklist for JWT revocation
const Redis = require('ioredis');
const redis = new Redis(process.env.REDIS_URL);

async function revokeToken(jti, expiresAt) {
  const ttl = Math.max(0, expiresAt - Math.floor(Date.now() / 1000));
  await redis.setex(`jwt:blacklist:${jti}`, ttl, '1');
}

async function isRevoked(jti) {
  return (await redis.exists(`jwt:blacklist:${jti}`)) === 1;
}
```

| Component | Breakdown |
|-----------|-----------|
| `jti` | Unique JWT ID claim. |
| `redis.setex` | Stores the blacklist entry with TTL. |
| `isRevoked` | Checks if the token is blacklisted. |

**Constraints and Limitations:**
- Every verification must check the blacklist, adding latency.
- The blacklist must be shared across all service instances (hence Redis).
- Blacklist entries must expire when the token would have expired naturally.

#### Annotated Code Example

```js
// jwt-revocation.js — Short-lived JWT with Redis blacklist
const { SignJWT, jwtVerify, generateKeyPair } = require('jose');
const Redis = require('ioredis');
const express = require('express');
const crypto = require('node:crypto');

(async () => {
  const { publicKey, privateKey } = await generateKeyPair('ES256');
  const redis = new Redis(process.env.REDIS_URL || 'redis://localhost:6379');
  const app = express();
  app.use(express.json());

  // Issue short-lived token
  app.post('/login', async (req, res) => {
    const jti = crypto.randomUUID();
    const expiresAt = Math.floor(Date.now() / 1000) + 900; // 15 minutes

    const token = await new SignJWT({ userId: req.body.userId })
      .setProtectedHeader({ alg: 'ES256' })
      .setIssuedAt()
      .setExpirationTime(expiresAt)
      .setJti(jti)
      .sign(privateKey);

    res.json({ token, expiresIn: 900 });
  });

  // Logout — revoke token
  app.post('/logout', async (req, res) => {
    const token = req.headers.authorization?.replace('Bearer ', '');
    const { payload } = await jwtVerify(token, publicKey, { algorithms: ['ES256'] });
    const ttl = payload.exp - Math.floor(Date.now() / 1000);
    if (ttl > 0) await redis.setex(`jwt:blacklist:${payload.jti}`, ttl, '1');
    res.json({ message: 'Logged out' });
  });

  // Protected route with blacklist check
  app.get('/protected', async (req, res) => {
    const token = req.headers.authorization?.replace('Bearer ', '');
    try {
      const { payload } = await jwtVerify(token, publicKey, { algorithms: ['ES256'] });
      const revoked = await redis.exists(`jwt:blacklist:${payload.jti}`);
      if (revoked) return res.status(401).json({ error: 'Token revoked' });
      res.json({ userId: payload.userId });
    } catch (err) {
      res.status(401).json({ error: 'Invalid token' });
    }
  });

  app.listen(3000, () => console.log('JWT revocation server on 3000'));
})();
```

**Expected Output (after logout, for `GET /protected` with the revoked token):**
```
{"error":"Token revoked"}
```

**Why this output:** The logout endpoint stores the `jti` in the Redis blacklist with a TTL matching the token's remaining lifetime. The protected route checks the blacklist before accepting the token, rejecting revoked tokens even if their signature is valid and they have not expired.

#### Real-World Cases

- **User logout:** Revoke the access token immediately rather than waiting for expiration.
- **Password change:** Revoke all existing tokens for the user.
- **Account compromise:** Revoke all tokens for the affected account.
- **Refresh token rotation:** Revoke the old refresh token when issuing a new one.

---

## Core Concept 4: Database Credentials

### Definitions

**Core Definition:** Database credential security involves isolating database user permissions to the minimum required for each workload and separating migration credentials from runtime application credentials.

**Technical Definition:** The principle of least privilege requires that each database user has only the permissions necessary for its function. A runtime application user needs `SELECT`, `INSERT`, `UPDATE`, and `DELETE` on application tables. A migration user needs `CREATE`, `ALTER`, and `DROP` permissions. These must be separate roles with separate credentials.

**Beginner-Friendly Explanation:** Think of a restaurant. The chef needs to cook (write to the database), the waiter needs to read orders (read from the database), but neither needs the keys to the safe (drop tables). A separate person (the migration user) comes in at closing time to change the menu (alter schema).

### Purposes

- To limit the blast radius of a compromised application credential.
- To prevent accidental schema modifications from runtime code.
- To enable separate rotation schedules for migration and runtime credentials.
- To comply with least-privilege requirements in security standards (SOC 2, PCI DSS).

### Sub-Feature 4.1: Least-Privilege Database Roles

#### Definitions

**Core Definition:** A least-privilege database role is a user account with only the permissions required to perform its specific function, and no more.

**Technical Definition:** In PostgreSQL, permissions are granted on tables, sequences, schemas, and functions. A runtime role might receive `SELECT, INSERT, UPDATE, DELETE` on application tables and `USAGE, SELECT` on sequences. A migration role receives `CREATE, ALTER, DROP` on schemas. The runtime role should not own the tables; ownership belongs to a separate role that is not used by the application.

**Beginner-Friendly Explanation:** Instead of giving the application the master key to the database, you give it a key that only opens the doors it needs. If the application is compromised, the attacker can only access the tables the application uses — not drop the entire database.

#### Purposes

- To limit the impact of a compromised application credential.
- To prevent runtime code from accidentally modifying the database schema.
- To enable independent rotation of runtime and migration credentials.

#### Syntax Rules and Structure

```sql
-- Create roles
CREATE ROLE app_runtime LOGIN PASSWORD 'runtime-password';
CREATE ROLE app_migration LOGIN PASSWORD 'migration-password';

-- Grant runtime permissions (least privilege)
GRANT USAGE ON SCHEMA public TO app_runtime;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_runtime;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_runtime;

-- Migration role has DDL permissions
GRANT ALL ON SCHEMA public TO app_migration;
GRANT ALL ON ALL TABLES IN SCHEMA public TO app_migration;
```

| Role | Permissions | Use Case |
|------|-------------|----------|
| `app_runtime` | DML only (SELECT, INSERT, UPDATE, DELETE) | Application queries |
| `app_migration` | DDL (CREATE, ALTER, DROP) | Schema migrations |

**Constraints and Limitations:**
- `GRANT ALL ON ALL TABLES` applies to existing tables only; new tables require additional grants.
- The runtime role should not own the tables; ownership should belong to a separate role.
- Connection poolers (PgBouncer) may require additional configuration for separate roles.

#### Annotated Code Example

```sql
-- setup-roles.sql — Separate runtime and migration roles
-- Run as a superuser or database owner

-- Create a role that owns the schema (not used by the application)
CREATE ROLE app_owner NOLOGIN;

-- Create runtime role (application uses this)
CREATE ROLE app_runtime LOGIN PASSWORD 'runtime-password';
GRANT USAGE ON SCHEMA public TO app_runtime;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_runtime;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_runtime;

-- Create migration role (CI/CD uses this)
CREATE ROLE app_migration LOGIN PASSWORD 'migration-password';
GRANT ALL ON SCHEMA public TO app_migration;
GRANT ALL ON ALL TABLES IN SCHEMA public TO app_migration;
GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO app_migration;

-- Ensure future tables grant runtime permissions automatically
ALTER DEFAULT PRIVILEGES FOR ROLE app_owner IN SCHEMA public
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_runtime;
ALTER DEFAULT PRIVILEGES FOR ROLE app_owner IN SCHEMA public
  GRANT USAGE, SELECT ON SEQUENCES TO app_runtime;
```

```js
// db.js — Runtime connection uses app_runtime role
const { Pool } = require('pg');
const pool = new Pool({
  connectionString: process.env.DATABASE_URL // postgres://app_runtime:...@host/db
});
module.exports = pool;
```

**Expected Output (runtime role attempting DDL):**
```
ERROR: permission denied for schema public
```

**Expected Output (runtime role performing DML):**
```
INSERT 0 1
```

**Why this output:** The runtime role has only DML permissions. Attempting `CREATE TABLE` or `DROP TABLE` fails with a permission error. Normal application queries (SELECT, INSERT, UPDATE, DELETE) succeed.

#### Real-World Cases

- **SaaS applications:** Each tenant's runtime role has access only to that tenant's schema.
- **Microservices:** Each service has its own runtime role with access only to its own tables.
- **CI/CD pipelines:** The migration role is used only during deployment, not at runtime.

---

### Sub-Feature 4.2: Dynamic Database Credentials

#### Definitions

**Core Definition:** Dynamic database credentials are short-lived credentials generated on demand by a secret manager (Vault, AWS Secrets Manager), eliminating long-lived database passwords.

**Technical Definition:** Vault's database secrets engine and AWS Secrets Manager's RDS integration generate unique database users with configurable TTLs. The application fetches credentials at startup (or on connection) and uses them for the credential's lifetime. When the credentials expire, the secret manager revokes the database user.

**Beginner-Friendly Explanation:** Instead of giving the application a permanent password, the secret manager creates a temporary user with a temporary password that expires after a set time. When the application needs to connect, it asks the secret manager for fresh credentials.

#### Purposes

- To eliminate long-lived database passwords entirely.
- To enable automatic revocation of credentials when they expire.
- To provide per-instance credentials that can be traced to a specific workload.
- To satisfy compliance requirements for short-lived credentials.

#### Syntax Rules and Structure

```js
// Vault dynamic database credentials
const creds = await vault.secret.db.generateCredentials('database', 'readonly').unwrap();
// creds = { username: 'v-app-readonly-abc123', password: '...', lease_id: '...', lease_duration: 3600 }
```

| Component | Breakdown |
|-----------|-----------|
| `generateCredentials` | Vault method for dynamic database credentials. |
| `lease_duration` | Credential lifetime in seconds. |
| `lease_id` | Used for renewal or revocation. |

**Constraints and Limitations:**
- Dynamic credentials require the secret manager to have administrative access to the database.
- Connection pools must be able to handle credential changes mid-lifecycle.
- Lease renewal must be handled to avoid mid-request credential expiration.

#### Annotated Code Example

```js
// dynamic-db.js — Dynamic database credentials with Vault
const { VaultClientV2 } = require('nanvc');
const { Pool } = require('pg');

const vault = new VaultClientV2({ endpoint: process.env.VAULT_ADDR });

async function createPool() {
  await vault.auth.loginWithAppRole({
    role_id: process.env.VAULT_ROLE_ID,
    secret_id: process.env.VAULT_SECRET_ID
  }).unwrap();

  const creds = await vault.secret.db.generateCredentials('database', 'readonly').unwrap();

  return new Pool({
    host: process.env.DB_HOST,
    database: 'app',
    user: creds.username,
    password: creds.password,
    max: 10
  });
}

(async () => {
  const pool = await createPool();
  const result = await pool.query('SELECT NOW()');
  console.log('Connected with dynamic credentials:', result.rows[0]);
})();
```

**Expected Output:**
```
Connected with dynamic credentials: { now: 2026-10-02T12:00:00.000Z }
```

**Why this output:** Vault generates a unique database user and password with a one-hour lease. The application uses these credentials to connect. When the lease expires, Vault revokes the user and the application must request new credentials.

#### Real-World Cases

- **PCI DSS compliance:** Dynamic credentials satisfy the requirement for unique, short-lived access.
- **Multi-tenant databases:** Each application instance gets its own database user, enabling per-instance audit.
- **Zero-trust architectures:** No long-lived credentials exist; every connection uses fresh, traceable credentials.

---

## Core Concept 5: Rotating Secrets

### Definitions

**Core Definition:** Secret rotation is the process of replacing a secret with a new value, updating all consumers, and revoking the old value, without causing application downtime.

**Technical Definition:** Zero-downtime rotation uses a dual-key overlap: the new secret is provisioned and distributed while the old secret remains valid, consumers are updated to use the new secret, and the old secret is revoked only after a drain window. For JWTs, rotation is implemented via a JWKS endpoint that publishes both old and new public keys; the issuer signs with the newest key while verifiers accept any key in the set. For database credentials, rotation uses connection pool health checks and transaction affinity to avoid disrupting in-flight queries.

**Beginner-Friendly Explanation:** Rotation is like changing the locks on your house. You don't change the lock while people are inside; you install the new lock alongside the old one, give everyone the new key, wait until everyone has used it, and then remove the old lock. Zero-downtime rotation ensures that no one is locked out during the change.

### Purposes

- To limit the lifetime of a secret, reducing the window of opportunity for misuse.
- To recover from a suspected secret compromise.
- To comply with security policies that mandate periodic rotation.
- To enable seamless credential updates without restarting the application.

### Sub-Feature 5.1: Zero-Downtime JWT Key Rotation

#### Definitions

**Core Definition:** Zero-downtime JWT key rotation involves publishing a new public key while the old key remains valid for verification, allowing tokens signed with either key to be accepted during the transition.

**Technical Definition:** A JWKS (JSON Web Key Set) endpoint publishes all active public keys with unique `kid` (key ID) values. The JWT header includes the `kid` of the signing key. Verifiers look up the key by `kid` and verify the signature. The issuer signs new tokens with the newest key. After the maximum token lifetime has elapsed, old keys are removed from the JWKS endpoint.

**Beginner-Friendly Explanation:** Think of a hotel with multiple master keys. When the hotel changes the master key, the old key still works for a while so guests who checked in before the change can still open their rooms. After those guests check out, the old key is deactivated.

#### Purposes

- To enable periodic key rotation without invalidating existing tokens.
- To recover from key compromise by revoking only the compromised key.
- To support multiple active keys during a transition period.
- To provide a standards-compliant JWKS endpoint for external verifiers.

#### Syntax Rules and Structure

```js
// JWKS endpoint with multiple keys
const jwks = {
  keys: [
    { kid: 'key-2026-01', alg: 'ES256', use: 'sig', kty: 'EC', crv: 'P-256', x: '...', y: '...' },
    { kid: 'key-2026-02', alg: 'ES256', use: 'sig', kty: 'EC', crv: 'P-256', x: '...', y: '...' }
  ]
};

app.get('/.well-known/jwks.json', (req, res) => res.json(jwks));
```

| Component | Breakdown |
|-----------|-----------|
| `kid` | Key identifier included in JWT header. |
| `jwks.keys` | Array of active public keys. |
| `use: 'sig'` | Key is for signature verification. |

**Constraints and Limitations:**
- Old keys must remain in the JWKS until all tokens signed with them have expired.
- Key IDs must be unique and unpredictable.
- The JWKS endpoint must be highly available; it is called by every verifier.

#### Annotated Code Example

```js
// jwt-rotation.js — Zero-downtime JWT key rotation
const { SignJWT, jwtVerify, generateKeyPair, exportJWK } = require('jose');
const express = require('express');

(async () => {
  // Generate two key pairs (in production, load from secret manager)
  const key1 = await generateKeyPair('ES256');
  const key2 = await generateKeyPair('ES256');

  const jwk1 = { ...(await exportJWK(key1.publicKey)), kid: 'key-2026-01', use: 'sig' };
  const jwk2 = { ...(await exportJWK(key2.publicKey)), kid: 'key-2026-02', use: 'sig' };

  const jwks = { keys: [jwk1, jwk2] };
  const signingKey = key2; // Newest key signs
  const signingKid = 'key-2026-02';

  const app = express();

  // JWKS endpoint
  app.get('/.well-known/jwks.json', (req, res) => res.json(jwks));

  // Issue token with newest key
  app.post('/login', async (req, res) => {
    const token = await new SignJWT({ userId: 'alice' })
      .setProtectedHeader({ alg: 'ES256', kid: signingKid })
      .setIssuedAt()
      .setExpirationTime('15m')
      .sign(signingKey.privateKey);
    res.json({ token });
  });

  // Verify with any key in the JWKS
  app.get('/protected', async (req, res) => {
    const token = req.headers.authorization?.replace('Bearer ', '');
    try {
      const { payload, protectedHeader } = await jwtVerify(token, async (header) => {
        const key = jwks.keys.find(k => k.kid === header.kid);
        if (!key) throw new Error('Unknown key');
        return key;
      });
      res.json({ userId: payload.userId, kid: protectedHeader.kid });
    } catch (err) {
      res.status(401).json({ error: 'Invalid token' });
    }
  });

  app.listen(3000, () => console.log('JWT rotation server on 3000'));
})();
```

**Expected Output (for `GET /protected` with a token signed by `key-2026-01` or `key-2026-02`):**
```
{"userId":"alice","kid":"key-2026-02"}
```

**Why this output:** The JWKS endpoint publishes both public keys. The verification function looks up the key by `kid` from the JWT header. Tokens signed with either key are accepted during the overlap period. After the old key is removed from the JWKS, only tokens signed with the new key are accepted.

#### Real-World Cases

- **OAuth 2.0 providers:** Rotate signing keys periodically and publish the JWKS for resource servers.
- **Microservices:** Each service verifies tokens using the shared JWKS endpoint.
- **Compliance:** PCI DSS and SOC 2 require documented key rotation procedures.

---

### Sub-Feature 5.2: Database Password Rotation Without Downtime

#### Definitions

**Core Definition:** Database password rotation without downtime uses a dual-connection-pool approach: a new pool is created with the new credentials, validated with a health check, and promoted for new queries while the old pool drains existing queries.

**Technical Definition:** The rotation workflow is: generate new credentials → create a new connection pool → validate the new pool with a health check → promote the new pool for new queries → keep the old pool alive for in-flight queries → retire the old pool after a drain timeout. NestJS-TypeORM-AWS-Connector implements this pattern with bounded rotation (at most one current and one retiring pool) and transaction affinity (queries stay on the generation where they started).

**Beginner-Friendly Explanation:** Imagine a restaurant changing its kitchen staff. The new chefs start cooking new orders while the old chefs finish the orders they already started. Once all old orders are served, the old chefs leave. No customer notices the change.

#### Purposes

- To rotate database credentials without interrupting in-flight queries.
- To validate new credentials before committing to them.
- To avoid connection pool errors during rotation.
- To provide a rollback path if the new credentials fail.

#### Syntax Rules and Structure

```js
// Dual-pool rotation pattern
let currentPool = await createPool(oldCredentials);
let retiringPool = null;

async function rotate(newCredentials) {
  const newPool = await createPool(newCredentials);
  await newPool.query('SELECT 1');    // Health check

  retiringPool = currentPool;          // Keep old pool for in-flight queries
  currentPool = newPool;               // New queries use new pool

  setTimeout(() => {
    retiringPool.end();                // Retire old pool after drain window
    retiringPool = null;
  }, 30000);                           // 30-second drain
}
```

| Component | Breakdown |
|-----------|-----------|
| `createPool` | Creates a new connection pool. |
| Health check | Validates the new pool before promotion. |
| `retiringPool` | Handles in-flight queries during drain. |
| `setTimeout` | Drain window before retiring the old pool. |

**Constraints and Limitations:**
- The drain window must be longer than the longest query.
- Transaction affinity must be maintained; queries within a transaction must use the same pool.
- The secret manager must revoke the old credentials after the drain window.

#### Annotated Code Example

```js
// db-rotation.js — Zero-downtime database password rotation
const { Pool } = require('pg');

let currentPool = null;
let retiringPool = null;

async function createPool(credentials) {
  return new Pool({
    host: process.env.DB_HOST,
    database: 'app',
    user: credentials.username,
    password: credentials.password,
    max: 10
  });
}

async function rotate(credentials) {
  const newPool = await createPool(credentials);

  // ✅ Health check before promotion
  await newPool.query('SELECT 1');

  // ✅ Promote new pool; keep old pool for in-flight queries
  retiringPool = currentPool;
  currentPool = newPool;

  // ✅ Retire old pool after drain window
  if (retiringPool) {
    setTimeout(async () => {
      await retiringPool.end();
      retiringPool = null;
    }, 30000);
  }
}

async function query(text, params) {
  return currentPool.query(text, params);
}

// Initial connection
(async () => {
  currentPool = await createPool({
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD
  });
  console.log('Connected to database');
})();

module.exports = { rotate, query };
```

**Expected Output (during rotation):**
```
Connected to database
(rotation proceeds without dropping existing connections)
```

**Why this output:** The `rotate` function creates a new pool, validates it with a health check, and promotes it for new queries. The old pool remains alive for 30 seconds to allow in-flight queries to complete. Queries in progress during rotation continue to use the old pool.

#### Real-World Cases

- **AWS RDS with Secrets Manager rotation:** Secrets Manager triggers a Lambda function that updates the database password; the application polls for the new secret and rotates its pool.
- **Vault dynamic credentials:** Vault issues new credentials on lease renewal; the application rotates its pool.
- **Compliance-driven rotation:** Rotating database passwords every 30, 60, or 90 days without downtime.

---

## Core Concept 6: Never Committing Secrets

### Definitions

**Core Definition:** Never committing secrets is the practice of preventing sensitive credentials from entering version control through defensive tooling such as pre-commit hooks, secret scanners, and CI/CD gates.

**Technical Definition:** Secret scanning tools (gitleaks, git-secrets, TruffleHog) detect secrets in source code, commit history, and CI pipelines. Pre-commit hooks run gitleaks or git-secrets before each commit, blocking commits that contain secrets. CI/CD pipelines run gitleaks as a hard gate, failing the build if any secret is detected. GitHub and GitLab also provide built-in secret scanning for supported platforms.

**Beginner-Friendly Explanation:** Secret scanning is like a security guard at the door of your repository. Before anyone can commit code, the guard checks their pockets for passwords and API keys. If a secret is found, the commit is blocked, and the developer is alerted.

### Purposes

- To prevent secrets from entering version control history, where they are extremely difficult to remove.
- To catch accidental commits before they leave the developer's machine.
- To provide a CI/CD gate that blocks pipelines if secrets are detected.
- To scan existing repositories for secrets that may have been committed previously.

### Sub-Feature 6.1: Pre-Commit Hooks with Gitleaks and git-secrets

#### Definitions

**Core Definition:** Pre-commit hooks are scripts that run automatically before each commit, scanning staged changes for secrets and blocking the commit if any are found.

**Technical Definition:** Gitleaks is a fast, open-source secret scanner written in Go. It can be installed as a pre-commit hook with `gitleaks protect --staged`. git-secrets, developed by AWS Labs, scans commits, commit messages, and merges for patterns like AWS keys. Both tools can be combined for maximum coverage.

**Beginner-Friendly Explanation:** A pre-commit hook is like a spell-checker for passwords. Before you save your work, the hook checks for anything that looks like a secret and warns you before you commit.

#### Purposes

- To catch secrets before they enter version control history.
- To provide immediate feedback to developers.
- To reduce the need for history scrubbing and credential rotation.

#### Syntax Rules and Structure

```bash
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
        entry: gitleaks protect --staged --verbose
        language: golang
        stages: [commit]

  - repo: https://github.com/awslabs/git-secrets
    rev: master
    hooks:
      - id: git-secrets
        entry: git secrets --scan
        language: system
        stages: [commit]
```

| Tool | Command | Purpose |
|------|---------|---------|
| Gitleaks | `gitleaks protect --staged` | Scans staged changes for secrets. |
| git-secrets | `git secrets --scan` | Scans commits for AWS keys and patterns. |

**Constraints and Limitations:**
- Pre-commit hooks can be bypassed with `git commit --no-verify`.
- Developers must install the hooks after cloning the repository.
- Some false positives may occur; a baseline file can suppress known non-secrets.

#### Annotated Code Example

```bash
# setup-hooks.sh — Install pre-commit hooks
#!/bin/bash

# Install gitleaks
brew install gitleaks

# Install pre-commit framework
pip install pre-commit

# Create pre-commit config
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
        entry: gitleaks protect --staged --verbose
        language: golang
        stages: [commit]
EOF

# Install hooks
pre-commit install

# Add baseline for known false positives
gitleaks detect --report-format json --report-path .gitleaks-baseline.json

echo "Pre-commit hooks installed."
```

**Expected Output (when a secret is detected):**
```
gitleaks............................................Failed
- hook id: gitleaks
- exit code: 1

Finding:     STRIPE_SECRET_KEY=sk_live_abc123...
File:        src/config.js
Line:        12
```

**Why this output:** The gitleaks hook scans the staged changes and finds a Stripe secret key. The commit is blocked, and the developer is shown the file, line number, and the secret that was detected.

#### Real-World Cases

- **Open-source repositories:** Pre-commit hooks prevent accidental commits of AWS keys, Stripe tokens, and database passwords.
- **Enterprise development:** Organisations mandate pre-commit hooks as part of their secure development lifecycle.
- **CI/CD pipelines:** The same gitleaks scan runs in the pipeline as a hard gate, catching secrets that bypassed local hooks.

---

### Sub-Feature 6.2: CI/CD Secret Scanning

#### Definitions

**Core Definition:** CI/CD secret scanning runs secret detection tools as part of the continuous integration pipeline, failing the build if any secret is detected in the repository or pipeline configuration.

**Technical Definition:** Gitleaks can be run in CI with `gitleaks detect --source .` to scan the entire repository (not just staged changes). GitHub Advanced Security provides built-in secret scanning for supported patterns. The CI job must be configured as a required check, and any detected secret must block the pipeline.

**Beginner-Friendly Explanation:** CI/CD scanning is a second line of defence. Even if a developer bypasses the pre-commit hook, the pipeline scans the entire repository and fails the build if a secret is found.

#### Purposes

- To catch secrets that bypassed local pre-commit hooks.
- To scan the full repository history, not just recent commits.
- To provide a centralised gate that cannot be bypassed by individual developers.
- To integrate with GitHub Advanced Security for organisation-wide scanning.

#### Syntax Rules and Structure

```yaml
# .github/workflows/security.yml
name: Secret Scan
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  gitleaks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for scanning
      - name: Run gitleaks
        run: gitleaks detect --source . --verbose
```

| Component | Breakdown |
|-----------|-----------|
| `fetch-depth: 0` | Fetches full history for scanning. |
| `gitleaks detect` | Scans the entire repository. |
| Required check | Blocks merge if secrets are found. |

**Constraints and Limitations:**
- Scanning full history can be slow for large repositories.
- False positives must be handled via a `.gitleaks.toml` configuration.
- GitHub Advanced Security is a paid feature for private repositories.

#### Annotated Code Example

```yaml
# .github/workflows/secret-scan.yml
name: Secret Scan
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  gitleaks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install gitleaks
        run: |
          wget https://github.com/gitleaks/gitleaks/releases/download/v8.18.0/gitleaks_8.18.0_linux_x64.tar.gz
          tar -xzf gitleaks_8.18.0_linux_x64.tar.gz
          sudo mv gitleaks /usr/local/bin/

      - name: Run gitleaks
        run: gitleaks detect --source . --verbose --redact
```

**Expected Output (when a secret is detected):**
```
Finding:     AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
File:        .env.production
Line:        5
Commit:      a1b2c3d4
Author:      dev@example.com
Date:        2026-10-01
```

**Why this output:** Gitleaks scans the full repository history and finds an AWS secret key in a committed file. The output includes the file, line, commit hash, author, and date, enabling the team to rotate the secret and remove it from history.

#### Real-World Cases

- **GitHub Actions:** The workflow runs on every push and pull request, blocking merges if secrets are found.
- **GitLab CI:** The same gitleaks scan runs as a CI job with `allow_failure: false`.
- **Jenkins:** Gitleaks runs as a build step and fails the build on detection.

---

## References

- OWASP Secrets Management Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
- Node.js Environment Variables Documentation — https://nodejs.org/api/process.html#processenv
- Node.js `--env-file` Documentation — https://nodejs.org/api/cli.html#--env-fileconfig
- Node.js Permissions Model — https://nodejs.org/api/permissions.html
- dotenv npm Package — https://www.npmjs.com/package/dotenv
- AWS SDK for JavaScript v3 — Secrets Manager — https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/client/secrets-manager/
- HashiCorp Vault Documentation — https://developer.hashicorp.com/vault/docs
- nanvc Vault Client — https://www.npmjs.com/package/nanvc
- node-vault npm Package — https://www.npmjs.com/package/node-vault
- Google Cloud Secret Manager Node.js Client — https://cloud.google.com/nodejs/docs/reference/secret-manager/latest
- jose npm Package — https://www.npmjs.com/package/jose
- RFC 7518 (JSON Web Algorithms) — https://datatracker.ietf.org/doc/html/rfc7518
- RFC 8037 (EdDSA for JOSE) — https://datatracker.ietf.org/doc/html/rfc8037
- OWASP JSON Web Token Cheat Sheet — https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
- PostgreSQL GRANT Documentation — https://www.postgresql.org/docs/current/sql-grant.html
- AWS Secrets Manager Rotation Documentation — https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html
- Gitleaks GitHub Repository — https://github.com/gitleaks/gitleaks
- git-secrets GitHub Repository — https://github.com/awslabs/git-secrets
- GitHub Secret Scanning — https://docs.github.com/en/code-security/secret-scanning
- Node.js Security Best Practices — https://nodejs.org/en/learn/getting-started/security-best-practices