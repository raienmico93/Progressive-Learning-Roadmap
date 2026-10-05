# Express-Specific Code-First Documentation Tools — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Code-first documentation tools are libraries and frameworks that generate OpenAPI specifications from existing Express route definitions, decorators, or JSDoc comments, rather than requiring developers to hand-write YAML or JSON specification files.

**Technical Definition:** Code-first documentation in Express operates on three architectural patterns. **Comment-based extraction** (`swagger-jsdoc`, `express-jsdoc-swagger`) parses JSDoc-style comments containing OpenAPI fragments from source files and assembles them into a specification. **Runtime validation middleware** (`express-openapi-validator`) consumes an existing OpenAPI document and validates incoming requests and outgoing responses against its schemas without modifying the route definitions. **Decorator-driven code generation** (`tsoa`) uses TypeScript decorators and type information on controller classes to generate both the OpenAPI specification and the Express route registration file, making TypeScript types the single source of truth.

**Beginner-Friendly Explanation:** Normally, you write an OpenAPI specification file by hand, and then you write your Express routes separately — and you have to keep them in sync. Code-first tools flip this: you write your routes (or controllers) as you normally would, add small annotations, and the tool generates the specification for you. This means your documentation is always up to date because it comes from the code itself.

### Key Characteristics

- **Single source of truth:** Documentation derives from the same code that handles requests, eliminating drift.
- **Comment-based tools are non-invasive:** `swagger-jsdoc` and `express-jsdoc-swagger` work with existing Express code without changing route structure.
- **Validation is bidirectional:** `express-openapi-validator` validates both requests and responses against the spec.
- **Decorator tools require TypeScript:** `tsoa` uses TypeScript decorators and type information to generate both the spec and the routes.
- **Tool selection depends on project constraints:** Comment-based tools for existing JavaScript apps; decorator tools for new TypeScript projects.

### Prerequisites

- **Node.js runtime** (v18 or higher; `swagger-jsdoc` v6 requires Node.js 20.x).
- **Express.js installed:** `npm install express`.
- **For swagger-jsdoc:** `npm install swagger-jsdoc swagger-ui-express`.
- **For express-openapi-validator:** `npm install express-openapi-validator`.
- **For express-jsdoc-swagger:** `npm install express-jsdoc-swagger`.
- **For tsoa:** `npm install tsoa express` and `npm install -D typescript @types/node @types/express`.

### Related Programming Areas

- **OpenAPI Specification:** The document format that all these tools produce or consume.
- **Interactive Documentation:** Swagger UI and ReDoc render the generated specifications.
- **Request/Response Validation:** Runtime schema validation based on OpenAPI definitions.
- **TypeScript Decorators:** The mechanism tsoa uses to extract metadata from controllers.
- **JSDoc:** The comment syntax used by swagger-jsdoc and express-jsdoc-swagger.

### Core Concepts

1. **swagger-jsdoc** — Writing OpenAPI specs inside Express route comments via JSDoc.
2. **express-openapi-validator** — Automatically validating requests/responses against your documentation.
3. **Auto-generating Specs from Express Routes** — tsoa and express-jsdoc-swagger.

---

## Core Concept 1: swagger-jsdoc

### Definitions

**Core Definition:** `swagger-jsdoc` is a library that reads JSDoc-annotated source code and generates an OpenAPI (Swagger) specification from `@openapi` or `@swagger` comment blocks embedded above Express route handlers. 

**Technical Definition:** `swagger-jsdoc` parses JSDoc-style comments containing OpenAPI fragments in YAML syntax. It accepts a configuration object with a `definition` (the base OpenAPI document structure) and an `apis` array of glob patterns pointing to files containing annotations. The library returns a validated OpenAPI specification object that can be passed to `swagger-ui-express` for rendering. Supported specifications include OpenAPI 3.x, Swagger 2.0, and AsyncAPI 2.0. 

**Beginner-Friendly Explanation:** Instead of maintaining a separate `swagger.json` file, you write your OpenAPI definitions right above your route handlers as comments. `swagger-jsdoc` reads those comments, assembles them into a complete specification, and hands it to Swagger UI for rendering. Your documentation lives right next to the code it describes, so they never drift apart.

### Purposes

- To generate OpenAPI specifications from JSDoc annotations embedded in Express route files.
- To keep documentation physically adjacent to the route handler it describes.
- To avoid maintaining a separate specification file that can drift out of sync.
- To integrate with Swagger UI for interactive documentation rendering.

### Syntax Rules and Structure

**Basic Configuration:**
```js
const swaggerJsdoc = require('swagger-jsdoc');

const options = {
  definition: {
    openapi: '3.0.0',
    info: { title: 'Hello World', version: '1.0.0' }
  },
  apis: ['./src/routes/*.js']  // Files containing annotations
};

const openapiSpecification = swaggerJsdoc(options);
```

**Route Annotation:**
```js
/**
 * @openapi
 * /:
 *   get:
 *     description: Welcome to swagger-jsdoc!
 *     responses:
 *       200:
 *         description: Returns a mysterious string.
 */
app.get('/', (req, res) => {
  res.send('Hello World!');
});
```

| Component | Breakdown |
|-----------|-----------|
| `definition` | Base OpenAPI object (info, servers, components). |
| `apis` | Glob patterns for files containing annotations. |
| `@openapi` / `@swagger` | Comment tags that mark the start of an OpenAPI fragment. |
| `failOnErrors` | When `true`, throws on parsing errors (default `false`). |

**Rules:**
- Annotations use YAML syntax inside JSDoc comment blocks.
- The `@openapi` tag is preferred over the legacy `@swagger` tag.
- The `apis` glob must point to files that contain annotations; the library parses them at build time.
- Use `failOnErrors: true` to catch malformed annotations during CI. 
- The generated specification is compatible with Swagger UI and ReDoc.
- `swagger-jsdoc` v6 requires Node.js 20.x or higher and is published as CommonJS. 

### Annotated Code Example

```js
// app.js
const express = require('express');
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const app = express();

// --- Swagger configuration ---
const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'User API',
      version: '1.0.0',
      description: 'API for managing users'
    },
    servers: [{ url: 'http://localhost:3000' }],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        }
      }
    }
  },
  apis: ['./routes/*.js']
};

const openapiSpec = swaggerJsdoc(options);

// Mount Swagger UI
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(openapiSpec));

app.listen(3000, () => {
  console.log('Server on 3000');
  console.log('Docs: http://localhost:3000/api-docs');
});
```

```js
// routes/users.js
const express = require('express');
const router = express.Router();

const users = [{ id: 1, name: 'Alice' }];

/**
 * @openapi
 * /users:
 *   get:
 *     summary: List all users
 *     tags: [Users]
 *     responses:
 *       200:
 *         description: List of users
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/User'
 */
router.get('/users', (req, res) => {
  res.json(users);
});

/**
 * @openapi
 * /users/{id}:
 *   get:
 *     summary: Get a user by ID
 *     tags: [Users]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: integer
 *     responses:
 *       200:
 *         description: User found
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/User'
 *       404:
 *         description: User not found
 */
router.get('/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: 'Not found' });
  res.json(user);
});

module.exports = router;
```

**Expected Output (rendered in Swagger UI):**
```
User API
  GET /users — List all users — Returns array of User
  GET /users/{id} — Get a user by ID — Returns User or 404
  Schemas: User
  Authorize: bearerAuth (JWT)
```

**Why this output:** The `swaggerJsdoc(options)` call reads the `options.definition` base document, then scans the `./routes/*.js` files for `@openapi` annotations. Each annotation contributes a path fragment to the final specification. Swagger UI renders the assembled spec with the `User` schema (defined in `components` or inferred from annotations) and the Bearer authentication scheme.

### Real-World Cases

- **Existing JavaScript Express apps:** Add documentation without restructuring routes or migrating to TypeScript.
- **Team documentation workflows:** Developers write annotations in the same file they edit, reducing documentation lag.
- **CI validation:** Use `failOnErrors: true` in a test to ensure annotations remain valid. 

---

## Core Concept 2: express-openapi-validator

### Definitions

**Core Definition:** `express-openapi-validator` is an Express middleware that automatically validates incoming API requests and outgoing API responses against an OpenAPI 3.x specification.

**Technical Definition:** `express-openapi-validator` is an unopinionated library that integrates with new and existing Express applications. It uses the OpenAPI document to validate request parameters, request bodies, response bodies, and security schemes. It supports OpenAPI 3.0.x and 3.1.x specifications. The validator provides configurable behaviours for request validation, response validation, security validation, and additional property handling (`ignore`, `remove`, `fail`). 

**Beginner-Friendly Explanation:** Once you have an OpenAPI specification, `express-openapi-validator` acts as a gatekeeper for every request and response. If a client sends a request that doesn't match the spec — missing required fields, wrong data types, invalid enum values — the middleware rejects it with a 400 error before it reaches your route handler. It can also validate that your responses match the spec, catching implementation bugs.

### Purposes

- To automatically validate requests against the OpenAPI specification.
- To automatically validate responses against the OpenAPI specification.
- To validate security schemes (API keys, Bearer tokens, OAuth 2.0) without writing custom middleware.
- To enforce request body schemas, query parameter types, and header requirements.
- To catch implementation drift between the spec and the actual API behaviour.

### Syntax Rules and Structure

**Basic Setup:**
```js
const OpenApiValidator = require('express-openapi-validator');

app.use(
  OpenApiValidator.middleware({
    apiSpec: './openapi.yaml',
    validateRequests: true,    // Validate requests (default: true)
    validateResponses: true,   // Validate responses (default: false)
    validateSecurity: true     // Validate security schemes (default: false)
  })
);
```

| Option | Default | Description |
|--------|---------|-------------|
| `apiSpec` | — | Path to the OpenAPI spec file. |
| `validateRequests` | `true` | Validate incoming requests. |
| `validateResponses` | `false` | Validate outgoing responses. |
| `validateSecurity` | `false` | Validate security schemes. |
| `coerceTypes` | `true` | Coerce query/path parameters to the types defined in the spec. |
| `removeAdditional` | `true` | Remove additional properties not defined in the spec. |
| `allErrors` | `false` | Return all validation errors instead of just the first. |

**Error Handling:**
```js
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({
    message: err.message,
    errors: err.errors
  });
});
```

**Rules:**
- The middleware must be mounted **before** route handlers so validation occurs first. 
- `validateResponses: true` requires that all responses match the spec — use with caution in development. 
- The validator returns a 400 for request validation failures and a 500 for response validation failures. 
- `coerceTypes: true` automatically converts string query parameters to the types defined in the spec (e.g., `"42"` → `42`). 
- OpenAPI 3.1 support was added in v5.4.0; earlier versions support only 3.0.x. 

### Annotated Code Example

```js
// app.js
const express = require('express');
const OpenApiValidator = require('express-openapi-validator');
const app = express();

app.use(express.json());

// Mount the validator
app.use(
  OpenApiValidator.middleware({
    apiSpec: './openapi.yaml',
    validateRequests: true,
    validateResponses: true,
    validateSecurity: true
  })
);

// Routes are defined normally — the validator intercepts before handlers
app.get('/users/:id', (req, res) => {
  // req.params.id is coerced to integer by the validator
  res.json({ id: req.params.id, name: 'Alice' });
});

app.post('/users', (req, res) => {
  // req.body is validated against the UserInput schema
  res.status(201).json({ id: 1, ...req.body });
});

// Error handler (must have 4 arguments)
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({
    message: err.message,
    errors: err.errors
  });
});

app.listen(3000, () => console.log('Validated server on 3000'));
```

**Expected Output (for a valid request):**
```json
{"id": 1, "name": "Alice"}
```

**Expected Output (for a request missing a required field):**
```json
{
  "message": "request.body should have required property 'name'",
  "errors": [
    {
      "path": ".body.name",
      "message": "should have required property 'name'",
      "errorCode": "required.openapi.validation"
    }
  ]
}
```

**Why this output:** The validator intercepts the request before the route handler. If the request body is missing the required `name` field (as defined in the OpenAPI spec), the validator returns a 400 error with a detailed message and error array. The route handler never executes. The error handler formats the response.

### Real-World Cases

- **API-first development:** Write the spec first, then implement routes — the validator enforces the contract.
- **Multi-team environments:** Frontend and backend teams agree on the spec; the validator catches integration mismatches early.
- **Security enforcement:** `validateSecurity: true` ensures that API key or Bearer token requirements defined in the spec are enforced at runtime. 

---

## Core Concept 3: Auto-Generating Specs from Express Routes (tsoa and express-jsdoc-swagger)

### Definitions

**Core Definition:** Auto-generation tools produce an OpenAPI specification directly from Express route definitions or TypeScript controllers, without requiring developers to write OpenAPI fragments in comments or separate files.

**Technical Definition:** `tsoa` is a framework with an integrated OpenAPI compiler that builds Node.js server-side applications using TypeScript. It uses decorators (`@Route`, `@Get`, `@Post`, `@Body`, `@Query`) on controller classes and TypeScript interfaces on models to generate both a valid OpenAPI specification and an Express route registration file (`routes.ts`). `express-jsdoc-swagger` is a lighter-weight alternative that scans JSDoc comments on Express endpoints and generates an OpenAPI 3.x specification and Swagger UI, without requiring TypeScript. 

**Beginner-Friendly Explanation:** Instead of writing documentation comments, you write TypeScript code with decorators, and tsoa generates the documentation for you. Your TypeScript types become the source of truth for the API's data models. `express-jsdoc-swagger` is a middle ground: it's like `swagger-jsdoc` but generates the spec and serves Swagger UI with less configuration.

### Purposes

- To eliminate manual specification writing by deriving the spec from code.
- To use TypeScript types as the single source of truth for API models.
- To generate Express route registration files automatically (tsoa).
- To provide runtime validation for Koa, Express, and Hapi services (tsoa).
- To offer a simpler alternative to swagger-jsdoc with automatic Swagger UI serving.

### Sub-Feature 3.1: tsoa

#### Syntax Rules and Structure

**tsoa Configuration (`tsoa.json`):**
```json
{
  "entryFile": "src/app.ts",
  "noImplicitAdditionalProperties": "throw-on-extras",
  "controllerPathGlobs": ["src/**/*Controller.ts"],
  "spec": {
    "outputDirectory": "build",
    "specVersion": 3
  },
  "routes": {
    "routesDir": "build"
  }
}
```

| Field | Description |
|-------|-------------|
| `entryFile` | The Express app entry point. |
| `controllerPathGlobs` | Glob patterns for controller files. |
| `noImplicitAdditionalProperties` | How to handle extra properties: `ignore`, `silently-remove-extras`, `throw-on-extras`. |
| `spec.specVersion` | OpenAPI version (`2` or `3`). |
| `routes.routesDir` | Output directory for the generated `routes.ts`. |

**Controller Definition:**
```typescript
import { Route, Get, Post, Body, Path, Query, Controller } from 'tsoa';

interface User {
  id: number;
  name: string;
  email: string;
}

interface UserInput {
  name: string;
  email: string;
}

@Route('users')
export class UsersController extends Controller {
  @Get()
  public async getUsers(@Query() limit?: number): Promise<User[]> {
    return [{ id: 1, name: 'Alice', email: 'alice@example.com' }];
  }

  @Get('{userId}')
  public async getUser(@Path() userId: number): Promise<User> {
    return { id: userId, name: 'Alice', email: 'alice@example.com' };
  }

  @Post()
  public async createUser(@Body() requestBody: UserInput): Promise<User> {
    return { id: 1, ...requestBody };
  }
}
```

**Express App Integration:**
```typescript
import express from 'express';
import { RegisterRoutes } from '../build/routes';

const app = express();
app.use(express.json());

RegisterRoutes(app);

app.listen(3000, () => console.log('tsoa server on 3000'));
```

**Generate Command:**
```bash
npx tsoa spec-and-routes
```

**Rules:**
- tsoa requires TypeScript and the `experimentalDecorators` compiler option. 
- Controllers extend `Controller` and use decorators to define routes and parameters. 
- TypeScript interfaces define the request and response models. 
- The generated `routes.ts` is registered with `RegisterRoutes(app)`. 
- tsoa supports Express, Hapi, and Koa; other frameworks can be added via Handlebar templates. 

#### Annotated Code Example

```typescript
// src/controllers/usersController.ts
import { Route, Get, Post, Body, Path, Query, Controller } from 'tsoa';

interface User {
  id: number;
  name: string;
  email: string;
}

interface UserInput {
  name: string;
  email: string;
}

@Route('users')
export class UsersController extends Controller {
  @Get()
  public async getUsers(
    @Query() limit: number = 20,
    @Query() page: number = 1
  ): Promise<User[]> {
    return [
      { id: 1, name: 'Alice', email: 'alice@example.com' },
      { id: 2, name: 'Bob', email: 'bob@example.com' }
    ].slice((page - 1) * limit, page * limit);
  }

  @Get('{userId}')
  public async getUser(@Path() userId: number): Promise<User> {
    return { id: userId, name: 'Alice', email: 'alice@example.com' };
  }

  @Post()
  public async createUser(@Body() requestBody: UserInput): Promise<User> {
    this.setStatus(201);
    return { id: 3, ...requestBody };
  }
}
```

**Generated OpenAPI specification (excerpt):**
```yaml
paths:
  /users:
    get:
      operationId: GetUsers
      parameters:
        - name: limit
          in: query
          required: false
          schema: { type: number, default: 20 }
        - name: page
          in: query
          required: false
          schema: { type: number, default: 1 }
      responses:
        '200':
          description: Ok
          content:
            application/json:
              schema:
                type: array
                items: { $ref: '#/components/schemas/User' }
    post:
      operationId: CreateUser
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/UserInput' }
      responses:
        '201':
          description: Created
          content:
            application/json:
              schema: { $ref: '#/components/schemas/User' }
```

**Expected Output (rendered in Swagger UI):**
```
Users
  GET /users — GetUsers
    Parameters: limit (number, default 20), page (number, default 1)
    Response: User[]
  GET /users/{userId} — GetUser
    Parameters: userId (number, path, required)
    Response: User
  POST /users — CreateUser
    Request Body: UserInput (required)
    Response: 201 — User
Schemas: User, UserInput
```

**Why this output:** tsoa reads the decorators and TypeScript types from the controller. `@Route('users')` defines the base path. `@Get()` and `@Post()` define operations. `@Query()`, `@Path()`, and `@Body()` define parameter locations. The TypeScript interfaces (`User`, `UserInput`) become OpenAPI schemas. The generated `routes.ts` registers these routes on the Express app, and the generated OpenAPI spec is consumed by Swagger UI.

### Sub-Feature 3.2: express-jsdoc-swagger

#### Syntax Rules and Structure

**Basic Setup:**
```js
const express = require('express');
const expressJSDocSwagger = require('express-jsdoc-swagger');

const app = express();

const options = {
  info: { version: '1.0.0', title: 'Albums store' },
  security: { BasicAuth: { type: 'http', scheme: 'basic' } },
  baseDir: __dirname,
  filesPattern: './**/*.js',
  swaggerUIPath: '/api-docs',
  exposeSwaggerUI: true,
  exposeApiDocs: false,
  apiDocsPath: '/v3/api-docs'
};

expressJSDocSwagger(app)(options);
```

**Route Annotation:**
```js
/**
 * GET /api/v1
 * @summary This is the summary of the endpoint
 * @return {object} 200 - success response
 */
app.get('/api/v1', (req, res) => res.json({ success: true }));
```

**Model Definition:**
```js
/**
 * A song type
 * @typedef {object} Song
 * @property {string} title.required - The title
 * @property {string} artist - The artist
 * @property {number} year - The year
 */
```

| Option | Description |
|--------|-------------|
| `info` | API metadata (version, title). |
| `security` | Security scheme definitions. |
| `baseDir` | Base directory for JSDoc files. |
| `filesPattern` | Glob pattern for files to scan. |
| `swaggerUIPath` | URL where Swagger UI is mounted. |
| `exposeSwaggerUI` | Whether to serve Swagger UI. |
| `exposeApiDocs` | Whether to expose the raw JSON spec. |

**Rules:**
- `express-jsdoc-swagger` scans files matching `filesPattern` for JSDoc annotations.
- The annotation syntax differs from `swagger-jsdoc`: it uses `@summary`, `@return`, and `@typedef` rather than raw OpenAPI YAML fragments.
- Swagger UI is served automatically at `swaggerUIPath` when `exposeSwaggerUI: true`.
- Models are defined with `@typedef` and properties with `@property`.
- The library generates OpenAPI 3.x specifications.

#### Annotated Code Example

```js
// app.js
const express = require('express');
const expressJSDocSwagger = require('express-jsdoc-swagger');

const app = express();

const options = {
  info: {
    version: '1.0.0',
    title: 'Albums Store',
    license: { name: 'MIT' }
  },
  security: {
    BasicAuth: { type: 'http', scheme: 'basic' }
  },
  baseDir: __dirname,
  filesPattern: './routes/**/*.js',
  swaggerUIPath: '/api-docs',
  exposeSwaggerUI: true,
  exposeApiDocs: true,
  apiDocsPath: '/v3/api-docs'
};

expressJSDocSwagger(app)(options);

app.listen(3000, () => {
  console.log('Server on 3000');
  console.log('Docs: http://localhost:3000/api-docs');
});
```

```js
// routes/albums.js
const express = require('express');
const router = express.Router();

/**
 * A song type
 * @typedef {object} Song
 * @property {string} title.required - The title
 * @property {string} artist - The artist
 * @property {number} year - The year
 */

/**
 * GET /api/v1/albums
 * @summary Get all albums
 * @tags Albums
 * @return {array<Song>} 200 - Albums list
 */
router.get('/api/v1/albums', (req, res) => {
  res.json([{ title: 'Abbey Road', artist: 'The Beatles', year: 1969 }]);
});

module.exports = router;
```

**Expected Output (rendered in Swagger UI):**
```
Albums Store
  GET /api/v1/albums — Get all albums
    Tags: Albums
    Response 200: array<Song>
Schemas: Song (title required, artist, year)
Security: BasicAuth
```

**Why this output:** The `@typedef` block defines the `Song` model with a required `title` property. The `@summary` and `@return` annotations on the route define the operation. `express-jsdoc-swagger` scans the `routes/` directory, extracts these annotations, and generates the OpenAPI spec and Swagger UI automatically.

### Real-World Cases

- **tsoa:** TypeScript-first teams building new APIs with decorators and type safety.
- **express-jsdoc-swagger:** JavaScript teams that want simpler annotation syntax than raw OpenAPI YAML.
- **Migration projects:** Adding documentation to existing Express apps with minimal code changes.
- **SDK generation:** tsoa-generated specs can feed OpenAPI Generator for client SDKs.

---

## References

- swagger-jsdoc on npm — https://www.npmjs.com/package/swagger-jsdoc
- swagger-jsdoc GitHub (Surnet) — https://github.com/Surnet/swagger-jsdoc
- express-openapi-validator GitHub (cdimascio) — https://github.com/cdimascio/express-openapi-validator
- express-openapi-validator Releases — https://github.com/cdimascio/express-openapi-validator/releases
- tsoa Getting Started Guide — https://tsoa-community.github.io/docs/getting-started.html
- tsoa Introduction — https://tsoa-community.github.io/docs/
- tsoa on OpenApi.tools — https://openapi.tools/#tsoa
- express-jsdoc-swagger GitHub (BRIKEV) — https://github.com/BRIKEV/express-jsdoc-swagger
- express-jsdoc-swagger Official Docs — https://brikev.github.io/express-jsdoc-swagger-docs/
- Express OpenAPI Extraction (swagger-jsdoc) — Specmatic Skills — https://github.com/specmatic/skills
- Implement API Documentation with Swagger/OpenAPI (GitHub Issue) — https://github.com/TevaLabs/Xelma-Backend/issues/36
- Validating Your API with OpenAPI (Steve Kinney) — https://stevekinney.com/courses/full-stack-typescript/validating-your-api-openapi
- express-openapi-validator Fork (npm) — https://www.npmjs.com/package/express-openapi-validator-fork
- How to Generate API Documentation (Express Discussion #6608) — https://github.com/expressjs/express/discussions/6608
- swagger-autogen (npm) — https://www.npmjs.com/package/swagger-autogen
- TSOA Alternatives (ThoughtWorks Technology Radar) — https://www.thoughtworks.com/radar/tools/typescript-openapi