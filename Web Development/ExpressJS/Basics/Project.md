# Project Setup

Setting up an Express.js project properly is the foundation for everything that follows. This guide covers initialization, dependency management, module systems, entry points, environment configuration, and the development tooling that keeps a project maintainable.

---

## 1. Initialize a Node.js Project

Every Node.js project begins with a `package.json` file — the manifest that describes the project, its dependencies, and its scripts.

### Creating the Project Directory

```bash
mkdir my-express-app
cd my-express-app
```

### Initializing `package.json`

**npm:**
```bash
npm init -y
```

**yarn:**
```bash
yarn init -y
```

**pnpm:**
```bash
pnpm init
```

The `-y` flag accepts all defaults. For more control, run without `-y` and answer the prompts.

### What `package.json` Contains

```json
{
  "name": "my-express-app",
  "version": "1.0.0",
  "description": "An Express.js API",
  "main": "src/server.js",
  "type": "module",
  "scripts": {
    "start": "node src/server.js",
    "dev": "node --watch src/server.js"
  },
  "keywords": ["express", "api"],
  "author": "Your Name",
  "license": "MIT"
}
```

Key fields:
- **`name`** — package identifier (lowercase, hyphens)
- **`version`** — semantic versioning (MAJOR.MINOR.PATCH)
- **`main`** — entry point for consumers
- **`type`** — module system (`"module"` for ESM, `"commonjs"` for CJS)
- **`scripts`** — named commands runnable via `npm run <name>`
- **`dependencies`** — packages needed at runtime
- **`devDependencies`** — packages needed only during development

---

## 2. Package Manager Configuration (npm, yarn, pnpm)

Node.js ships with **npm**, but several package managers offer different trade-offs.

### Comparison

| Feature | npm | yarn | pnpm |
|---|---|---|---|
| **Bundled with Node.js** | Yes | No | No |
| **Lockfile** | `package-lock.json` | `yarn.lock` | `pnpm-lock.yaml` |
| **Install speed** | Moderate | Fast | Fastest |
| **Disk usage** | Duplicates per project | Duplicates per project | Hard links, shared store |
| **Workspaces** | Yes (v7+) | Yes | Yes (excellent) |
| **Strictness** | Permissive | Permissive | Strict (no phantom deps) |
| **Best for** | Default, compatibility | Legacy projects | Monorepos, disk efficiency |

### Installing Dependencies

| Action | npm | yarn | pnpm |
|---|---|---|---|
| Install all | `npm install` | `yarn` | `pnpm install` |
| Add production dep | `npm install express` | `yarn add express` | `pnpm add express` |
| Add dev dep | `npm install -D nodemon` | `yarn add -D nodemon` | `pnpm add -D nodemon` |
| Remove dep | `npm uninstall express` | `yarn remove express` | `pnpm remove express` |
| Run script | `npm run dev` | `yarn dev` | `pnpm dev` |

### Choosing a Package Manager

- **npm** — default, zero setup, broadest compatibility
- **yarn** — mature, good for existing projects
- **pnpm** — fastest, most disk-efficient, best for monorepos

**Recommendation for new projects:** Use **pnpm** for speed and disk efficiency, or **npm** for simplicity and ubiquity.

---

## 3. Install Express

Express is installed as a **production dependency** because it is required at runtime.

```bash
# npm
npm install express

# yarn
yarn add express

# pnpm
pnpm add express
```

### Express 5

Express 5.x is the current major version, with improvements to async error handling, path routing, and Promise support.

```bash
pnpm add express@5
```

### Verify Installation

```bash
# Check installed version
npm list express

# Or inspect package.json
cat package.json
```

---

## 4. Managing Production Dependencies vs. Development Tools

Dependencies fall into two categories.

### Production Dependencies (`dependencies`)

Packages required for the application to **run in production**.

**Examples:**
- `express` — web framework
- `dotenv` — environment variable loading
- `helmet` — security headers
- `cors` — cross-origin resource sharing
- `zod` — input validation
- `pg` / `mongoose` / `prisma` — database clients

```bash
pnpm add express helmet cors zod
```

### Development Dependencies (`devDependencies`)

Packages needed only during **development, testing, and building**.

**Examples:**
- `nodemon` — auto-restart server
- `eslint` — linting
- `prettier` — formatting
- `typescript` — type checking
- `@types/node`, `@types/express` — type definitions
- `jest` / `vitest` — testing

```bash
pnpm add -D nodemon eslint prettier typescript @types/node @types/express
```

### Why the Distinction Matters

| Aspect | `dependencies` | `devDependencies` |
|---|---|---|
| Installed in production | Yes | No |
| Installed with `--production` | Yes | No |
| Impact on bundle/image size | Larger | None |
| Security surface | Larger | Smaller |

**Best practice:** Keep `devDependencies` lean — every package is a potential vulnerability and adds install time.

---

## 5. Configure `package.json`

A well-configured `package.json` centralizes project metadata and scripts.

### Essential Fields

```json
{
  "name": "my-express-app",
  "version": "1.0.0",
  "description": "REST API built with Express",
  "main": "src/server.js",
  "type": "module",
  "engines": {
    "node": ">=20.0.0"
  },
  "scripts": {
    "start": "node src/server.js",
    "dev": "node --watch src/server.js",
    "lint": "eslint src/**/*.js",
    "format": "prettier --write src/**/*.js",
    "test": "node --test"
  },
  "dependencies": {
    "express": "^5.0.0"
  },
  "devDependencies": {
    "eslint": "^9.0.0",
    "prettier": "^3.0.0"
  }
}
```

### Key Fields Explained

| Field | Purpose |
|---|---|
| **`type`** | `"module"` for ESM, `"commonjs"` for CJS |
| **`main`** | Entry point when imported as a package |
| **`engines`** | Node.js version requirement |
| **`scripts`** | Named commands for common tasks |
| **`private`** | Prevents accidental publish to npm |

### Script Naming Conventions

| Script | Purpose |
|---|---|
| `start` | Run the application (production) |
| `dev` | Run in development mode |
| `build` | Compile/transpile if needed |
| `test` | Run tests |
| `lint` | Run linter |
| `format` | Run formatter |

**Note:** `start` is special — `npm start` works without `run`.

---

## 6. ECMAScript Modules (`"type": "module"`) vs. Legacy CommonJS

Node.js supports two module systems. Choosing one is a foundational decision.

### CommonJS (Legacy, Default Without `type`)

```javascript
// Import
const express = require('express');
const { readFile } = require('fs');

// Export
module.exports = { myFunction };
exports.myFunction = myFunction;
```

**Characteristics:**
- Synchronous loading
- `require()` and `module.exports`
- Default when `package.json` lacks `"type"`
- Works in all Node.js versions

### ES Modules (Modern, `"type": "module"`)

```javascript
// Import
import express from 'express';
import { readFile } from 'fs/promises';

// Export
export function myFunction() { }
export default myFunction;
```

**Characteristics:**
- Asynchronous loading (with top-level `await`)
- `import` and `export`
- Enabled by `"type": "module"` in `package.json`
- Standard JavaScript module syntax
- Compatible with browsers

### Enabling ESM

Add `"type": "module"` to `package.json`:

```json
{
  "type": "module"
}
```

Then all `.js` files in the package are treated as ES modules.

### File Extension Rules

| Extension | Behavior |
|---|---|
| `.js` | ESM if `"type": "module"`, else CJS |
| `.mjs` | Always ESM |
| `.cjs` | Always CJS |

### ESM vs. CJS Comparison

| Aspect | CommonJS | ES Modules |
|---|---|---|
| Syntax | `require` / `module.exports` | `import` / `export` |
| Loading | Synchronous | Asynchronous |
| Top-level `await` | No | Yes |
| Tree-shaking | Difficult | Supported |
| Browser compatibility | No (without bundler) | Yes |
| `__dirname` / `__filename` | Available | Not available (use `import.meta.url`) |
| Dynamic import | `require()` | `import()` |
| Standard | Legacy | TC39 standard |

### Why ESM Is Preferred for New Projects

> "Prefer ES Modules (ESM) for new projects (`import`/`export`)."

**Reasons:**
- **Standard** — ESM is the official JavaScript module system
- **Tree-shaking** — bundlers can eliminate dead code
- **Top-level `await`** — simplifies async initialization
- **Browser alignment** — same syntax in browsers and Node
- **Future-proof** — Node.js is moving toward ESM as default

**The article on modern Node.js + Express setup confirms:**
> "Instead of using CommonJS: `const express = require('express');` We use modern ES Modules: `import express from 'express';` In `package.json`: `"type": "module"` Node 18+ handles this beautifully."

### Common ESM Pitfalls

**`__dirname` and `__filename` do not exist in ESM:**

```javascript
// CJS
console.log(__dirname);

// ESM alternative
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

**JSON imports require assertions:**

```javascript
// ESM JSON import
import data from './data.json' with { type: 'json' };
```

**`.js` extensions required in relative imports:**

```javascript
// ESM
import { helper } from './utils/helper.js';  // extension required

// CJS
const { helper } = require('./utils/helper');  // extension optional
```

### Custom Run Scripts for Developer Workflows

Configure scripts to match your module system:

```json
{
  "type": "module",
  "scripts": {
    "start": "node src/server.js",
    "dev": "node --watch src/server.js",
    "build": "tsc",
    "test": "node --test"
  }
}
```

### When to Use CommonJS

- **Legacy codebase** — migrating to ESM is a large effort
- **Dependencies that only support CJS** — rare but possible
- **Simple scripts** — CJS is still fine for one-off tools

**Recommendation:** Use **ESM for new projects**. If working with an existing CJS codebase, migrate incrementally.

---

## 7. Create Application Entry Point

The entry point is where Node.js starts executing. A clean separation of concerns improves testability and maintainability.

### Best Practices: `app.js` vs. `server.js`

Separate the **app** (Express configuration) from the **server** (network binding):

| File | Responsibility |
|---|---|
| **`app.js`** | Create Express app, register middleware, mount routes |
| **`server.js`** | Import app, bind to port, start listening |

> "Keep `app.js` (build the app) separate from `server.js` (start it). Your tests can import app without opening a port."

### `app.js` — Build the App

```javascript
// src/app.js
import express from 'express';
import helmet from 'helmet';
import cors from 'cors';
import routes from './routes/index.js';
import { errorHandler } from './middleware/error.js';

const app = express();

// Middleware
app.use(helmet());
app.use(cors());
app.use(express.json());

// Routes
app.use('/api', routes);

// Error handler (must be LAST)
app.use(errorHandler);

export default app;
```

### `server.js` — Start the Server

```javascript
// src/server.js
import app from './app.js';

const PORT = process.env.PORT || 3000;

const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

// Graceful shutdown
process.on('SIGTERM', () => {
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
});
```

### Why This Separation Matters

| Benefit | Explanation |
|---|---|
| **Testability** | Tests import `app` without binding a port |
| **Flexibility** | Same app can be served by different servers |
| **Clarity** | Configuration vs. startup is explicit |
| **Graceful shutdown** | Server-level concerns (SIGTERM) live in `server.js` |

### Alternative Entry Point Names

| File | Common Use |
|---|---|
| `index.js` | Default npm entry; often re-exports or starts server |
| `server.js` | Explicit server startup |
| `app.js` | Express app configuration |
| `main.js` | Alternative naming |

### Folder Structure

A scalable structure recommended for 2026:

```
src/
├── config/          # env, db connection, clients
├── models/          # schemas
├── routes/          # URL wiring only
├── controllers/     # request/response glue
├── services/        # business logic
├── middleware/      # auth, validation, error handler
├── utils/           # pure helpers
├── app.js           # express app (no listen)
└── server.js        # starts the server
```


---

## 8. Start the Server

### Basic Startup

```javascript
const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Validating Local Host Bindings

By default, `app.listen(PORT)` binds to all interfaces. For security, bind explicitly:

```javascript
// Bind to localhost only (local development)
app.listen(PORT, '127.0.0.1', () => {
  console.log(`Server running at http://127.0.0.1:${PORT}`);
});

// Bind to all interfaces (accessible on the network)
app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on all interfaces, port ${PORT}`);
});
```

| Binding | Behavior | Use Case |
|---|---|---|
| `localhost` / `127.0.0.1` | Local machine only | Local development |
| `0.0.0.0` | All network interfaces | Containers, servers |
| Specific IP | That interface only | Multi-homed hosts |

**Security note:** Binding to `0.0.0.0` in development exposes your app to the local network. Prefer `127.0.0.1` locally.

### Dynamic Port Allocation

Cloud platforms (Heroku, Render, Railway, Google App Engine, AWS) assign a port dynamically via `process.env.PORT`.

> "Your server must use the port specified by `process.env.PORT`, an environment variable defined by the App Engine runtime."

```javascript
const PORT = process.env.PORT || 3000;

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server listening on port ${PORT}`);
});
```

**Why `0.0.0.0` matters in containers:** Inside Docker or a cloud container, binding to `localhost` makes the app unreachable from outside the container.

### Port Conflicts

If the port is in use, Node.js throws `EADDRINUSE`. Handle it gracefully:

```javascript
server.on('error', (err) => {
  if (err.code === 'EADDRINUSE') {
    console.error(`Port ${PORT} is already in use`);
    process.exit(1);
  }
  throw err;
});
```

---

## 9. Development vs. Production Environments

Node.js uses the `NODE_ENV` environment variable to signal the runtime environment.

### `NODE_ENV` Values

| Value | Purpose |
|---|---|
| `development` | Local development; verbose logging, hot reload, detailed errors |
| `production` | Deployed application; optimized, minimal logging, generic errors |
| `test` | Automated testing; isolated, deterministic |

### Setting `NODE_ENV`

**Via command line (native flag):**

```bash
NODE_ENV=production node src/server.js
```

**Via `package.json` scripts:**

```json
{
  "scripts": {
    "dev": "NODE_ENV=development node --watch src/server.js",
    "start": "NODE_ENV=production node src/server.js"
  }
}
```

**Via `.env` file (development):**

```
NODE_ENV=development
PORT=3000
```

### Using `NODE_ENV` in Code

```javascript
if (process.env.NODE_ENV === 'production') {
  app.use(helmet({ contentSecurityPolicy: true }));
} else {
  app.use(morgan('dev'));  // verbose logging
}
```

### Environment-Specific Configuration

```javascript
// config/env.js
import 'dotenv/config';

export const env = {
  port: process.env.PORT || 3000,
  nodeEnv: process.env.NODE_ENV || 'development',
  dbUrl: process.env.DATABASE_URL,
  jwtSecret: process.env.JWT_SECRET,
};

// Fail fast if required variables are missing
if (!env.dbUrl) throw new Error('DATABASE_URL is required');
```


---

## 10. Securely Parsing Variables Using `.env` Files

Environment variables should never be hard-coded. Use `.env` files for local development and secret managers for production.

### Using `dotenv`

```bash
pnpm add dotenv
```

Create `.env` at the project root:

```
NODE_ENV=development
PORT=3000
DATABASE_URL=postgres://user:pass@localhost:5432/app
JWT_SECRET=super-secret-key
```

Load it **as early as possible**:

```javascript
// At the very top of server.js, before any other imports
import 'dotenv/config';
```


### Native Node.js `.env` Support

Node.js 20.6+ reads `.env` files natively — no `dotenv` dependency required:

> "Node 20.6+ reads `.env` files natively — no `dotenv` dependency required."

```bash
node --env-file=.env src/server.js
```

Or in `package.json`:

```json
{
  "scripts": {
    "dev": "node --watch --env-file=.env src/server.js"
  }
}
```

### Security Best Practices

#### 1. Never Commit `.env`

Add to `.gitignore` **before the first commit**:

```
.env
.env.*
!.env.example
```


**If already committed:** Rotate the secrets and scrub history — deleting the file in a later commit does not remove it from history.

#### 2. Provide `.env.example`

Commit a template with keys but no values:

```
# .env.example
NODE_ENV=development
PORT=3000
DATABASE_URL=
JWT_SECRET=
```


#### 3. Validate Required Variables at Startup

Fail fast if variables are missing:

```javascript
const required = ['DATABASE_URL', 'JWT_SECRET'];
const missing = required.filter((k) => !process.env[k]);

if (missing.length > 0) {
  throw new Error(`Missing env vars: ${missing.join(', ')}`);
}
```


#### 4. Never Log `process.env`

Dumping the environment leaks every secret at once.

#### 5. Use Secret Managers in Production

For staging and production, use a managed secret store:

- **AWS Secrets Manager**
- **HashiCorp Vault**
- **Google Secret Manager**
- **Azure Key Vault**
- **Doppler**

> "For anything sensitive, a flat file on disk is a weak place to store production credentials. Managed secret stores ... give you rotation, audit trails, and access control that a text file cannot."

#### 6. Exclude `.env` from Docker Builds

Add to `.dockerignore`:

```
.env
.env.*
```


#### 7. Environment-Specific Files

```
.env                # shared defaults (committed if non-sensitive)
.env.development    # local development
.env.production     # production (never committed)
.env.local          # local overrides (never committed)
```


---

## 11. Useful Development Tooling

### 11.1 Nodemon

**Nodemon** watches files and restarts the server on changes.

```bash
pnpm add -D nodemon
```

**Configuration (`nodemon.json`):**

```json
{
  "watch": ["src"],
  "ext": "js,json",
  "ignore": ["node_modules", "dist"],
  "exec": "node src/server.js"
}
```


**Script:**

```json
{
  "scripts": {
    "dev": "nodemon"
  }
}
```

#### Nodemon vs. Native `--watch`

Node.js 18+ includes a native `--watch` flag:

```bash
node --watch src/server.js
```

> "Developers can utilize the built-in watch flag instead of nodemon."

| Aspect | Nodemon | Native `--watch` |
|---|---|---|
| **Dependency** | External package | Built into Node.js |
| **Memory usage** | ~200 MB overhead reported | Minimal |
| **Restart speed** | 2–3 seconds (reported) | Faster |
| **Configuration** | `nodemon.json` | CLI flags |
| **File filtering** | `ext`, `ignore` | `--watch-path`, `--watch-preserve-output` |
| **Debugging** | `--inspect` passthrough | `--inspect` works |
| **Best for** | Complex watch rules | Simple projects, fewer deps |

> "Native Node.js watch is built into the runtime, no extra process. Better integration with the Node.js event loop. Less overhead from monitoring file systems. No need for additional dependencies."

**Recommendation for 2026:** Use **native `--watch`** for simple projects. Use **nodemon** if you need fine-grained control (watching specific directories, custom exec commands).

**Native watch with specific paths:**

```bash
node --watch-path=./src --watch-path=./tests src/server.js
```


---

### 11.2 Debuggers

#### VS Code Debugger

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch Server",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/src/server.js",
      "envFile": "${workspaceFolder}/.env",
      "console": "integratedTerminal",
      "restart": true,
      "skipFiles": [
        "<node_internals>/**",
        "${workspaceFolder}/node_modules/**"
      ]
    }
  ]
}
```


**Key settings:**
- **`envFile`** — loads `.env` automatically
- **`restart: true`** — restarts debugger if the process crashes
- **`skipFiles`** — prevents stepping into Node internals and dependencies
- **`console: "integratedTerminal"`** — allows `stdin` interaction

#### Attach to Running Process

For debugging a server started externally:

```json
{
  "name": "Attach to Process",
  "type": "node",
  "request": "attach",
  "port": 9229,
  "restart": true
}
```

Start the server with the inspector:

```bash
node --inspect src/server.js
```

#### Chrome DevTools

1. Start Node with `--inspect` or `--inspect-brk`:
   ```bash
   node --inspect-brk src/server.js
   ```
2. Open `chrome://inspect` in Chrome
3. Click **"Open dedicated DevTools for Node"**
4. Set breakpoints, inspect variables, profile performance

---

### 11.3 Linters

**ESLint** catches bugs and enforces code quality.

```bash
pnpm add -D eslint
npx eslint --init
```

**Configuration (`eslint.config.js` — flat config):**

```javascript
import js from '@eslint/js';

export default [
  js.configs.recommended,
  {
    languageOptions: {
      ecmaVersion: 2024,
      sourceType: 'module',
    },
    rules: {
      'no-unused-vars': 'warn',
      'no-console': 'off',
      'prefer-const': 'error',
    },
  },
];
```

**Script:**

```json
{
  "scripts": {
    "lint": "eslint src/**/*.js",
    "lint:fix": "eslint src/**/*.js --fix"
  }
}
```

#### Strict Type-Checking with TypeScript

TypeScript adds static types, catching errors before runtime:

```bash
pnpm add -D typescript @types/node @types/express
npx tsc --init
```

**`tsconfig.json` (strict):**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"]
}
```


**Scripts:**

```json
{
  "scripts": {
    "build": "tsc",
    "start": "node dist/server.js",
    "dev": "node --watch --experimental-strip-types src/server.ts"
  }
}
```

**TypeScript benefits:**
- Catches type errors at compile time
- Better IDE autocomplete and refactoring
- Self-documenting code via types
- Safer refactoring

---

### 11.4 Formatters

**Prettier** enforces consistent code style.

```bash
pnpm add -D prettier
```

**Configuration (`.prettierrc`):**

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2
}
```

**`.prettierignore`:**

```
node_modules
dist
coverage
```

**Script:**

```json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  }
}
```

**ESLint + Prettier integration:**

```bash
pnpm add -D eslint-config-prettier
```

```javascript
// eslint.config.js
import js from '@eslint/js';
import prettier from 'eslint-config-prettier';

export default [
  js.configs.recommended,
  prettier,  // disables ESLint rules that conflict with Prettier
];
```

**VS Code integration:**

Install the **Prettier** and **ESLint** extensions, then add to `.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  }
}
```

---

## Summary Table

| Topic | Key Points |
|---|---|
| **Initialize project** | `npm init -y`, `yarn init -y`, or `pnpm init` |
| **Package managers** | npm (default), yarn (mature), pnpm (fast, disk-efficient) |
| **Install Express** | `pnpm add express` (production dependency) |
| **Dependencies** | `dependencies` (runtime) vs. `devDependencies` (build/test) |
| **`package.json`** | `type`, `main`, `engines`, `scripts`, `private` |
| **Module system** | `"type": "module"` for ESM (modern); default CJS (legacy) |
| **Entry point** | `app.js` (build app) + `server.js` (start server) |
| **Server startup** | `app.listen(PORT)`, bind to `127.0.0.1` or `0.0.0.0` |
| **Dynamic port** | `process.env.PORT \|\| 3000` |
| **Environments** | `NODE_ENV=development` / `production` / `test` |
| **`.env` files** | `dotenv` or native `--env-file`; never commit; validate at startup |
| **Nodemon** | Auto-restart; compare with native `--watch` |
| **Debugger** | VS Code `launch.json` with `envFile`, `skipFiles` |
| **Linter** | ESLint (flat config); TypeScript for strict types |
| **Formatter** | Prettier; integrate with ESLint and VS Code |

---

## Key Takeaways

1. **Initialize with a package manager** — `pnpm init` for speed, `npm init -y` for ubiquity.
2. **Express is a production dependency** — install it with `pnpm add express`.
3. **Separate `dependencies` from `devDependencies`** — keep production images lean.
4. **Prefer `"type": "module"` (ESM)** for new projects — it is the standard, supports top-level `await`, and aligns with browsers.
5. **Separate `app.js` (build the app) from `server.js` (start the server)** — improves testability and clarity.
6. **Use `process.env.PORT`** for dynamic port allocation in cloud environments.
7. **Bind carefully** — `127.0.0.1` locally, `0.0.0.0` in containers.
8. **Set `NODE_ENV`** to control logging, error detail, and optimization.
9. **Never commit `.env`** — use `.gitignore`, provide `.env.example`, and validate required variables at startup.
10. **Use secret managers** (Vault, AWS Secrets Manager) in production — `.env` files are for local development only.
11. **Use native `--watch`** instead of nodemon for simpler projects; nodemon for complex watch rules.
12. **Configure VS Code debugging** with `launch.json` and `envFile`.
13. **Lint with ESLint**, format with **Prettier**, and consider **TypeScript** for type safety.

---

Would you like me to continue with the next topic — **Express Routing**, **Express Middleware**, or **Request and Response Objects**? I can format the next section in the same style.