# API Documentation — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** **API documentation** is the technical reference that describes how to use and integrate with an API — its available endpoints, request and response formats, authentication methods, error handling, rate limits, and lifecycle policies. It is both a **human-facing artifact** (guides, tutorials, reference pages) and a **machine-readable contract** (typically an OpenAPI document) from which tooling can generate interactive references, client SDKs, server stubs, and automated tests.

**Technical Definition.** API documentation in the modern ecosystem is anchored on the **OpenAPI Specification (OAS)**, a standard, programming-language-agnostic interface description for HTTP APIs. An OpenAPI document defines the API’s **paths and operations**, **parameters**, **request bodies**, **responses**, **schemas**, **security schemes**, and **metadata** in JSON or YAML. This document is the **single source of truth** from which rendering tools (e.g., Swagger UI), code generators, validators, and linters derive their behavior. The documentation surface itself comprises multiple layers: an OpenAPI description, a rendered interactive reference, task-oriented guides, runnable code samples, authentication instructions, error catalogs, and a changelog.

**Beginner-Friendly Explanation.** API documentation is **the instruction manual for your API**. If your API is a machine that developers need to operate, the documentation tells them which buttons to press, what inputs the machine expects, what outputs it produces, and what happens when something goes wrong. Good documentation gets a developer from “never seen this API” to “successful first call” in minutes. Great documentation also shows working code, explains every error, makes authentication obvious, and lets developers **try requests directly from the docs**.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Specification-first** | The OpenAPI document is the single source of truth; reference docs are generated from it, not written by hand. |
| **Interactive** | Swagger UI reads the OpenAPI document and renders a testable interface where developers can send requests. |
| **Multi-audience** | Documentation serves evaluators, first-time integrators, experienced maintainers, and even AI agents. |
| **Machine-readable** | The OpenAPI document enables code generation, mock servers, contract tests, and linting. |
| **Living** | Documentation is validated, linted, and published automatically on every API change; manual drift is eliminated. |
| **Layered** | It combines reference (lookup), tutorials (learning), how-to guides (tasks), and explanations (concepts). |

---

### Prerequisites

- **HTTP fundamentals**: methods, status codes, headers, request/response model.
- **REST fundamentals**: resources, endpoints, representations, statelessness.
- **JSON and YAML**: the serialization formats used by OpenAPI documents.
- **Node.js and npm**: for the `swagger-jsdoc` and `swagger-ui-express` toolchain.
- **Express.js basics**: routes, middleware, and app setup (for the Express integration examples).
- **A tool to test APIs**: `curl`, Postman, or the Swagger UI playground.

---

### Related Programming Areas

- **API design**: the OpenAPI document is both a design artifact and a documentation artifact.
- **Code generation**: OpenAPI documents drive client SDK and server stub generation.
- **Contract testing**: OpenAPI schemas are used to validate that implementations match the documented contract.
- **API gateways**: API Management platforms import OpenAPI documents to configure routing, policies, and portals.
- **Developer experience (DX)**: documentation quality directly affects adoption, support volume, and integration time.

---

### Core Concepts / Features

The following six sections cover the requested concepts. Each follows the uniform structure: definitions, purposes, syntax rules, annotated executable examples with expected outputs, and real-world cases.

---

## 1. OpenAPI

### Definitions

**Core Definition.** The **OpenAPI Specification (OAS)** is a standard, programming-language-agnostic interface description language for HTTP APIs. It allows both humans and computers to discover and understand the capabilities of a service without access to source code, additional documentation, or network traffic inspection.

**Technical Definition.** OAS defines a **root document** with required fields including `openapi` (the version string, e.g., `3.1.1`), `info` (metadata: title, version, description), `paths` (the available endpoints and operations), and optional but essential sections for `components` (reusable schemas, parameters, responses, security schemes), `servers`, `security`, `tags`, and `externalDocs`. Operations are described by HTTP method under each path, with `parameters`, `requestBody`, and `responses`. Schemas are described using the **JSON Schema vocabulary** (OAS 3.1 aligns fully with JSON Schema 2020-12). The specification uses normative keywords (MUST, SHOULD, MAY) as defined by BCP 14.

**Beginner-Friendly Explanation.** OpenAPI is **a standard language for describing your API in a file**. You write a YAML or JSON file that lists every endpoint, what inputs it accepts, what it returns, and how authentication works. Tools can read this file to generate documentation, client libraries, and tests. It is like a **blueprint** that both humans and machines can understand. The blueprint is called the **OpenAPI document**, and it is the single source of truth for everything else.

### Purposes

- **To provide a machine-readable contract** that tools can use for documentation generation, code generation, mocking, and testing.
- **To eliminate guesswork** in calling a service by formally describing inputs, outputs, and behaviors.
- **To enable a single source of truth** for documentation, SDKs, and contract tests, preventing drift.
- **To support language-agnostic tooling** across JavaScript, Python, Java, Go, and other ecosystems.
- **To document authentication, errors, and edge cases** alongside the happy path.

### Syntax Rules and Structure

**General syntax (minimal OpenAPI 3.1 document in YAML).**

```yaml
openapi: 3.1.1
info:
  title: Example API
  version: 1.0.0
  description: A minimal OpenAPI document.
paths:
  /users:
    get:
      summary: List users
      responses:
        '200':
          description: A list of users
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      required: [id, name]
      properties:
        id:
          type: integer
        name:
          type: string
```

**Component breakdown.**

- `openapi` — the OAS version (`3.1.1`); the `major.minor` portion designates the feature set.
- `info` — API metadata: `title` and `version` are required; `description` is optional but essential for humans.
- `paths` — the relative paths and their operations; each operation is keyed by HTTP method (`get`, `post`, etc.).
- `responses` — keyed by status code; each requires a `description`.
- `content` — the media type and schema of the response body.
- `components/schemas` — reusable data models referenced via `$ref`.

**OpenAPI structure rules.**

| Rule | Guidance |
|---|---|
| **`info.title` and `info.version` are required** | Every OpenAPI document must identify itself. |
| **Paths start with `/`** | Paths are relative to the `servers` URL. |
| **Status codes are strings** | `'200'`, `'404'`, not numeric `200`. |
| **Use `$ref` for reusability** | Reference `components` instead of duplicating schemas. |
| **OAS 3.1 aligns with JSON Schema 2020-12** | Use standard JSON Schema keywords; `nullable` is replaced by `type: [string, "null"]`. |

**Constraints and limitations.**

- **The document can become large**; split reusable components and consider multiple files with `$ref`.
- **The specification defines structure, not quality**; a valid OpenAPI document can still be poorly written.
- **Tooling compatibility varies**; OAS 3.1 features (e.g., JSON Schema alignment, `webhooks`) are not supported by all older tools.

### Annotated Complete Step by Step Code Examples

#### Example 1: A minimal OpenAPI 3.1 document for a CRUD resource

```yaml
# openapi.yaml
openapi: 3.1.1
info:
  title: User API
  version: 1.0.0
  description: A simple API for managing users.

paths:
  /users:
    get:
      summary: List all users
      operationId: listUsers
      tags: [Users]
      responses:
        '200':
          description: A list of users
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
    post:
      summary: Create a user
      operationId: createUser
      tags: [Users]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUser'
      responses:
        '201':
          description: User created
          headers:
            Location:
              schema:
                type: string
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'

components:
  schemas:
    User:
      type: object
      required: [id, name, email]
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
        email:
          type: string
          format: email
    CreateUser:
      type: object
      required: [name, email]
      properties:
        name:
          type: string
        email:
          type: string
          format: email
```

**Expected Output (conceptual):**
```
A valid OpenAPI 3.1 document that tools can parse.
It describes GET /users and POST /users with schemas, status codes, and a Location header.
```

**Why this output occurs.** The document follows the required OAS structure: `openapi`, `info`, `paths`, and `components`. The `$ref` references keep schemas DRY. The `post` operation includes a `requestBody` and a `201` response with a `Location` header, accurately describing the CRUD Create operation.

---

#### Example 2: Adding security and error documentation

```yaml
openapi: 3.1.1
info:
  title: Secure User API
  version: 1.0.0

security:
  - bearerAuth: []

paths:
  /users/{id}:
    get:
      summary: Get a user
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '401':
          description: Missing or invalid token
          content:
            application/problem+json:
              schema:
                $ref: '#/components/schemas/Problem'
        '404':
          description: User not found
          content:
            application/problem+json:
              schema:
                $ref: '#/components/schemas/Problem'

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  schemas:
    User:
      type: object
      properties:
        id: { type: integer }
        name: { type: string }
    Problem:
      type: object
      required: [type, title, status]
      properties:
        type: { type: string, format: uri }
        title: { type: string }
        status: { type: integer }
        detail: { type: string }
```

**Expected Output (conceptual):**
```
The document declares global bearer authentication and documents 401 and 404 error responses using RFC 9457 Problem Details.
```

**Why this output occurs.** The top-level `security` requirement applies bearer authentication to all operations. The `401` and `404` responses reference a reusable `Problem` schema, making error handling explicit and consistent. This demonstrates how OpenAPI documents authentication and failures alongside success cases.

---

### Real-World Cases

- **Stripe API**: publishes an OpenAPI description that powers its reference documentation and client libraries.
- **GitHub REST API**: provides an OpenAPI description used by tooling and documentation generators.
- **Google APIs**: use OpenAPI-compatible discovery documents to drive client generation and API consoles.

---

## 2. Swagger

### Definitions

**Core Definition.** **Swagger** is the original name of the specification that became **OpenAPI**, and today refers to the **family of open-source and commercial tools** built around the OpenAPI Specification by SmartBear — including Swagger UI, Swagger Editor, Swagger Codegen, and Swagger Hub.

**Technical Definition.** The Swagger specification was created in 2010 by Tony Tam at Wordnik. In 2015, SmartBear donated the specification to the newly formed OpenAPI Initiative under the Linux Foundation. On **1 January 2016**, the specification was renamed the **OpenAPI Specification (OAS)**. Since then, “OpenAPI” refers to the **specification**, while “Swagger” refers to the **tooling ecosystem**. The two names are often used interchangeably in practice, but conflating them causes confusion: a “Swagger file” is an OpenAPI document; “Swagger UI” is a renderer; “Swagger Editor” is an authoring tool.

**Beginner-Friendly Explanation.** Swagger is **the original name for OpenAPI, and also the name of the tools around it**. Think of it this way: **OpenAPI is the blueprint format**, and **Swagger is the brand of tools** that help you write, view, and use blueprints. When someone says “Swagger file,” they usually mean an OpenAPI document. When they say “Swagger UI,” they mean the interactive documentation page. The specification was renamed to OpenAPI in 2016, but the tool names kept the Swagger brand.

### Purposes

- **To provide a recognizable tool ecosystem** for authoring, validating, rendering, and generating code from OpenAPI documents.
- **To clarify the distinction between the specification (OpenAPI) and its tooling (Swagger)** so teams use terminology correctly.
- **To support the full API lifecycle** — design (Swagger Editor), document (Swagger UI), generate (Swagger Codegen), and collaborate (Swagger Hub).

### Syntax Rules and Structure

Swagger is a **tool family**, not a syntax. Its components include:

| Tool | Purpose |
|---|---|
| **Swagger Editor** | Browser-based editor for writing and validating OpenAPI documents. |
| **Swagger UI** | Renders an OpenAPI document as interactive documentation. |
| **Swagger Codegen** | Generates client SDKs and server stubs from OpenAPI documents. |
| **Swagger Hub** | Collaborative platform for designing and hosting OpenAPI documents. |
| **Swagger Parser** | JavaScript library for parsing and validating OpenAPI documents. |

**Terminology rules.**

- **“OpenAPI document”** — the specification file (YAML or JSON).
- **“Swagger UI”** — the interactive renderer.
- **“Swagger Editor”** — the authoring environment.
- **“Swagger file”** — colloquial, but technically an OpenAPI document.

**Constraints and limitations.**

- **“Swagger” is sometimes used to mean the old 2.0 specification**, which is materially different from OpenAPI 3.x; always specify the version.
- **Tool names do not track specification versions**; Swagger UI supports multiple OpenAPI versions, but its name has not changed.

### Annotated Complete Step by Step Code Examples

#### Example 1: Distinguishing OpenAPI (specification) from Swagger (tools)

```yaml
# This is an OpenAPI document (specification).
openapi: 3.1.1
info:
  title: Example
  version: 1.0.0
paths: {}
```

```javascript
// This uses Swagger UI (a tool) to render the OpenAPI document.
const SwaggerUI = require('swagger-ui');
SwaggerUI({
  url: '/openapi.yaml',   // the OpenAPI document
  dom_id: '#swagger-ui',  // the DOM node to render into
});
```

**Expected Output (conceptual):**
```
The OpenAPI document is the data.
Swagger UI is the renderer that turns the data into an interactive page.
```

**Why this output occurs.** The `openapi` field marks the file as an OpenAPI document. Swagger UI reads that document and renders it. The terminology distinction matters because “Swagger” alone is ambiguous: it could mean the old specification or the tooling ecosystem.

---

### Real-World Cases

- **Swagger UI** is embedded in thousands of APIs’ `/docs` or `/api-docs` routes, rendering OpenAPI documents as interactive references.
- **Swagger Editor** is used by API teams to write and validate OpenAPI documents before committing them.
- **Swagger Codegen** generates client SDKs for languages such as Java, Python, and TypeScript from OpenAPI documents.

---

## 3. Swagger UI

### Definitions

**Core Definition.** **Swagger UI** is an open-source tool that reads an **OpenAPI Specification document** and generates a **visual, interactive documentation page** where developers can explore endpoints and send test requests directly from the browser.

**Technical Definition.** Swagger UI is a dependency-free collection of HTML, JavaScript, and CSS assets that dynamically render an OpenAPI document. It parses the document, displays operations grouped by tags, shows parameters and schemas, and provides “Try it out” functionality that sends real HTTP requests. It is distributed as three npm modules: `swagger-ui` (for bundled applications), `swagger-ui-dist` (dependency-free, for server-side serving), and `swagger-ui-react` (as a React component). It can also be embedded via a standalone HTML page, a Docker image, or a CDN.

**Beginner-Friendly Explanation.** Swagger UI is **the interactive documentation page** developers see when they visit `/docs` or `/api-docs`. It reads your OpenAPI document and turns it into a clickable, testable interface. You can expand an endpoint, read its description, see the expected inputs, fill in values, and click “Try it out” to send a real request. It is the most visible part of API documentation — the part developers actually interact with.

### Purposes

- **To visualize an OpenAPI document** as browsable, human-readable documentation.
- **To enable interactive testing** — developers can send requests from the browser without writing code.
- **To group operations by tags** so the documentation follows product concepts rather than code modules.
- **To embed documentation directly in the API server** via middleware or standalone hosting.

### Syntax Rules and Structure

**General syntax (initialization in a browser).**

```javascript
import SwaggerUI from 'swagger-ui';

SwaggerUI({
  url: '/openapi.yaml',       // URL or path to the OpenAPI document
  dom_id: '#swagger-ui',      // DOM element to render into
  deepLinking: true,          // enable deep links to operations
  presets: [SwaggerUI.presets.apis],
});
```

**Component breakdown.**

- `url` — the location of the OpenAPI document (JSON or YAML).
- `dom_id` — the CSS selector of the element where Swagger UI renders.
- `deepLinking` — enables URL fragments for individual operations.
- `presets` — configuration presets (e.g., `apis` for a standard API explorer).

**Swagger UI configuration rules.**

| Rule | Guidance |
|---|---|
| **The OpenAPI document must be valid** | Swagger UI cannot render an invalid document meaningfully. |
| **Use `swagger-ui` for bundled apps** | Use `swagger-ui-dist` only for server-side or no-bundler contexts. |
| **Configure `validatorUrl` for production** | Point it to your own validator or disable it to avoid sending spec URLs to a third party. |
| **Use `deepLinking` for shareable links** | Each operation gets a URL fragment that can be bookmarked. |

**Constraints and limitations.**

- **Swagger UI is a renderer, not an authoring tool**; it does not fix poor descriptions or missing examples.
- **OAS 3.1 support depends on the Swagger UI version**; verify compatibility before relying on 3.1 features.
- **“Try it out” sends real requests**; in production, ensure it is secured or disabled where appropriate.

### Annotated Complete Step by Step Code Examples

#### Example 1: Standalone HTML page with Swagger UI via CDN

```html
<!DOCTYPE html>
<html>
<head>
  <title>API Docs</title>
  <link rel="stylesheet" href="https://unpkg.com/swagger-ui-dist/swagger-ui.css" />
</head>
<body>
  <div id="swagger-ui"></div>

  <script src="https://unpkg.com/swagger-ui-dist/swagger-ui-bundle.js"></script>
  <script>
    SwaggerUIBundle({
      url: "/openapi.yaml",
      dom_id: "#swagger-ui",
    });
  </script>
</body>
</html>
```

**Expected Output (conceptual):**
```
A fully rendered interactive documentation page at the HTML file's URL.
The page displays all endpoints from /openapi.yaml with expandable operations and "Try it out" buttons.
```

**Why this output occurs.** The HTML loads the Swagger UI CSS and JavaScript bundle from a CDN, then calls `SwaggerUIBundle` with the OpenAPI document URL. Swagger UI fetches the document, parses it, and renders the operations into the `#swagger-ui` div. No build step is required.

---

#### Example 2: Customizing Swagger UI behavior

```javascript
SwaggerUIBundle({
  url: "/openapi.yaml",
  dom_id: "#swagger-ui",
  deepLinking: true,
  displayRequestDuration: true,
  filter: true,
  tryItOutEnabled: true,
  presets: [SwaggerUIBundle.presets.apis],
});
```

**Expected Output (conceptual):**
```
A documentation page with deep links per operation, request duration display, a filter/search box, and enabled "Try it out" functionality.
```

**Why this output occurs.** Each option changes a specific rendering behavior: `deepLinking` adds URL fragments, `displayRequestDuration` shows timing, `filter` adds a search box, and `tryItOutEnabled` pre-enables the interactive request panel. These options are passed directly to the Swagger UI renderer.

---

### Real-World Cases

- **Petstore example**: Swagger UI’s default demo uses the Petstore OpenAPI document to showcase rendering and interaction.
- **Internal API portals**: Companies host Swagger UI at `/api-docs` to give internal teams a self-service reference.
- **Public API references**: Many public APIs embed Swagger UI directly in their documentation sites.

---

## 4. `swagger-jsdoc`

### Definitions

**Core Definition.** **`swagger-jsdoc`** is a Node.js library that reads **JSDoc-annotated source code** and generates a validated **OpenAPI specification** in JSON or YAML. It allows developers to document API endpoints directly in the code using `@openapi` (or `@swagger`) comments, keeping documentation close to the implementation.

**Technical Definition.** `swagger-jsdoc` scans the files specified in its `apis` option, extracts JSDoc comments containing `@openapi` or `@swagger` blocks, parses them as YAML, and merges them with the base `definition` object. It returns the assembled specification as a JavaScript object. It supports OpenAPI 3.x, Swagger 2, and AsyncAPI 2.0, and can be configured with `failOnErrors: true` to throw on validation failures, making it suitable for unit tests that enforce documentation validity.

**Beginner-Friendly Explanation.** `swagger-jsdoc` lets you **write your API documentation right above your routes**. Instead of maintaining a separate YAML file that drifts from the code, you write a comment like `@openapi` above each Express route, and `swagger-jsdoc` collects all those comments into a complete OpenAPI document. It keeps documentation **close to the code**, reducing the chance that the docs and the implementation disagree.

### Purposes

- **To keep documentation co-located with route handlers**, reducing drift between code and docs.
- **To generate a complete OpenAPI document** from inline JSDoc annotations without maintaining a separate file.
- **To enforce documentation validity in CI** via `failOnErrors: true` and unit tests.
- **To support multiple specification formats** (OpenAPI 3.x, Swagger 2, AsyncAPI 2.0).

### Syntax Rules and Structure

**General syntax (installation and basic usage).**

```bash
npm install swagger-jsdoc --save
```

```javascript
const swaggerJsdoc = require('swagger-jsdoc');

const options = {
  definition: {
    openapi: '3.1.1',
    info: {
      title: 'Hello World',
      version: '1.0.0',
    },
  },
  apis: ['./src/routes/*.js'],   // files containing annotations
};

const openapiSpecification = swaggerJsdoc(options);
```

**Component breakdown.**

- `definition` — the base OpenAPI object; must include `openapi` and `info`.
- `apis` — glob patterns for files containing `@openapi` or `@swagger` annotations.
- `failOnErrors` — optional; when `true`, throws on parsing or validation errors.
- The returned `openapiSpecification` is a validated OpenAPI object.

**Annotation syntax (inline).**

```javascript
/**
 * @openapi
 * /users:
 *   get:
 *     summary: List users
 *     responses:
 *       200:
 *         description: A list of users
 */
app.get('/users', (req, res) => { /* ... */ });
```

**`swagger-jsdoc` rules.**

| Rule | Guidance |
|---|---|
| **Use `@openapi` (recommended) or `@swagger`** | Both are recognized; `@openapi` is preferred for OpenAPI 3.x. |
| **The annotation body is YAML** | Indentation matters; malformed YAML breaks generation. |
| **Base `definition` must include `openapi` and `info`** | These are required by the OAS. |
| **Use `failOnErrors: true` in CI** | Catch invalid annotations before they reach production. |

**Constraints and limitations.**

- **The generated document is only as good as the annotations**; incomplete JSDoc blocks produce incomplete documentation.
- **Large codebases can produce unwieldy annotations**; consider splitting into multiple files and using `$ref`.
- **`swagger-jsdoc` is in maintenance mode**; the project recommends `zod-openapi` or `@asteasolutions/zod-to-openapi` for new projects in some contexts.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generating an OpenAPI document from Express routes

```javascript
// app.js
const express = require('express');
const swaggerJsdoc = require('swagger-jsdoc');

const app = express();

/**
 * @openapi
 * /users:
 *   get:
 *     summary: List users
 *     responses:
 *       200:
 *         description: A list of users
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/User'
 */
app.get('/users', (req, res) => {
  res.json([{ id: 1, name: 'Alice' }]);
});

const options = {
  definition: {
    openapi: '3.1.1',
    info: { title: 'User API', version: '1.0.0' },
    components: {
      schemas: {
        User: {
          type: 'object',
          properties: {
            id: { type: 'integer' },
            name: { type: 'string' },
          },
        },
      },
    },
  },
  apis: ['./app.js'],
};

const openapiSpec = swaggerJsdoc(options);
console.log(JSON.stringify(openapiSpec, null, 2));
```

**Expected Output (abbreviated):**
```json
{
  "openapi": "3.1.1",
  "info": { "title": "User API", "version": "1.0.0" },
  "paths": {
    "/users": {
      "get": {
        "summary": "List users",
        "responses": {
          "200": {
            "description": "A list of users",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": { "$ref": "#/components/schemas/User" }
                }
              }
            }
          }
        }
      }
    }
  },
  "components": {
    "schemas": {
      "User": {
        "type": "object",
        "properties": {
          "id": { "type": "integer" },
          "name": { "type": "string" }
        }
      }
    }
  }
}
```

**Why this output occurs.** `swagger-jsdoc` scans `app.js`, extracts the `@openapi` YAML block, parses it, and merges it with the base `definition`. The result is a complete OpenAPI document that includes the path, operation, response, and reusable schema. This document can then be passed to Swagger UI for rendering.

---

#### Example 2: Validating documentation in CI

```javascript
// test/openapi.test.js
const swaggerJsdoc = require('swagger-jsdoc');

const options = {
  failOnErrors: true,   // throw on parsing/validation errors
  definition: {
    openapi: '3.1.1',
    info: { title: 'User API', version: '1.0.0' },
  },
  apis: ['./src/routes/*.js'],
};

test('OpenAPI document is valid', () => {
  expect(() => swaggerJsdoc(options)).not.toThrow();
});
```

**Expected Output (conceptual):**
```
PASS  test/openapi.test.js
  ✓ OpenAPI document is valid
```

**Why this output occurs.** With `failOnErrors: true`, `swagger-jsdoc` throws when any annotation is malformed or the resulting document fails validation. The unit test catches that throw and fails the build, preventing invalid documentation from being published.

---

### Real-World Cases

- **Express-based APIs** annotate route handlers with `@openapi` blocks and generate the OpenAPI document at startup.
- **CI pipelines** run `swagger-jsdoc` with `failOnErrors: true` to enforce documentation validity on every pull request.
- **Legacy migrations** use `swagger-jsdoc` to incrementally document existing Express routes without rewriting them.

---

## 5. `swagger-ui-express`

### Definitions

**Core Definition.** **`swagger-ui-express`** is an Express middleware that serves **Swagger UI**-generated API documentation from within an Express application, based on an OpenAPI document (typically a `swagger.json` file or a `swagger-jsdoc`-generated object).

**Technical Definition.** `swagger-ui-express` pulls the Swagger UI assets from `swagger-ui-dist` and mounts them as middleware on an Express route. It provides two handlers: `swaggerUi.serve` (serves the static assets) and `swaggerUi.setup(document, options)` (renders the UI for the given OpenAPI document). It supports customization through options such as `explorer` (shows the Swagger Explorer bar), `swaggerOptions` (passed to the Swagger UI client), and `customCss` (injected into the page). The result is **living documentation** hosted directly by the API server.

**Beginner-Friendly Explanation.** `swagger-ui-express` lets your Express API **host its own documentation page**. You add two lines of middleware, and suddenly `/api-docs` shows the Swagger UI with all your endpoints. It is the bridge between your OpenAPI document and a live, interactive page served by your own server. Developers do not need a separate documentation site; the docs live right next to the API.

### Purposes

- **To serve interactive API documentation directly from the Express application**, eliminating a separate hosting step.
- **To combine `swagger-jsdoc` and Swagger UI** in a single pipeline: generate the spec, then render it.
- **To customize the documentation experience** through `explorer`, `swaggerOptions`, and `customCss`.
- **To provide living documentation** that updates automatically when the API changes.

### Syntax Rules and Structure

**General syntax (basic setup).**

```javascript
const express = require('express');
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

const app = express();

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument));
```

**Component breakdown.**

- `swaggerUi.serve` — serves the Swagger UI static assets.
- `swaggerUi.setup(swaggerDocument)` — renders the UI for the given OpenAPI document.
- `app.use('/api-docs', ...)` — mounts the documentation at the `/api-docs` path.

**Integration with `swagger-jsdoc`.**

```javascript
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const swaggerSpec = swaggerJsdoc(options);

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

**`swagger-ui-express` configuration rules.**

| Rule | Guidance |
|---|---|
| **Use `swaggerUi.serve` before `setup`** | `serve` provides the static assets; `setup` renders the page. |
| **For Express Router, use `router.use` + `router.get`** | See the router example below. |
| **Pass the OpenAPI object, not a path** | `setup` expects a document object, not a URL string. |
| **Use `customCss` for branding** | Inject custom styles to match your organization’s design. |

**Constraints and limitations.**

- **`swagger-ui-express` serves documentation to anyone who can reach the route**; secure or restrict it in production if the API is internal.
- **The middleware does not validate the OpenAPI document**; use `swagger-jsdoc` with `failOnErrors` or a separate validator.
- **Version compatibility between `swagger-ui-express` and `swagger-ui-dist`** should be managed with a lock file.

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic Express setup with a static OpenAPI document

```javascript
// app.js
const express = require('express');
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

const app = express();

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument));

app.listen(3000, () => {
  console.log('Docs available at http://localhost:3000/api-docs');
});
```

**Expected Output:**
```
Docs available at http://localhost:3000/api-docs
```

**Why this output occurs.** `swaggerUi.serve` serves the Swagger UI assets, and `swaggerUi.setup(swaggerDocument)` renders the UI for the provided OpenAPI document. Visiting `/api-docs` shows the interactive documentation page.

---

#### Example 2: Full pipeline with `swagger-jsdoc`

```javascript
// app.js
const express = require('express');
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const app = express();

/**
 * @openapi
 * /health:
 *   get:
 *     summary: Health check
 *     responses:
 *       200:
 *         description: Service is healthy
 */
app.get('/health', (req, res) => res.json({ status: 'ok' }));

const options = {
  definition: {
    openapi: '3.1.1',
    info: { title: 'Health API', version: '1.0.0' },
  },
  apis: ['./app.js'],
};

const swaggerSpec = swaggerJsdoc(options);

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));

app.listen(3000);
```

**Expected Output (conceptual):**
```
GET /api-docs renders the interactive documentation with the /health endpoint.
```

**Why this output occurs.** `swagger-jsdoc` generates the OpenAPI specification from the inline `@openapi` annotation and the base `definition`. That specification is passed to `swagger-ui-express`, which renders it as interactive documentation. The entire pipeline — annotation, generation, rendering — runs in a single Express application.

---

#### Example 3: Express Router integration

```javascript
const router = require('express').Router();
const swaggerUi = require('swagger-ui-express');
const swaggerDocument = require('./swagger.json');

router.use('/api-docs', swaggerUi.serve);
router.get('/api-docs', swaggerUi.setup(swaggerDocument));
```

**Expected Output (conceptual):**
```
The documentation is served on the router's /api-docs path.
```

**Why this output occurs.** With Express Router, `router.use` mounts the static assets, and `router.get` handles the documentation page. This pattern is useful when the documentation should be scoped to a specific router rather than the global app.

---

### Real-World Cases

- **Internal admin APIs**: serve Swagger UI at `/api-docs` so internal teams have a self-service reference without a separate documentation deployment.
- **Microservices**: each service hosts its own Swagger UI, and a central portal aggregates them via links.
- **Prototyping**: combine `swagger-jsdoc` and `swagger-ui-express` to generate and render documentation in a single Express app during development.

---

## 6. API Documentation Best Practices

### Definitions

**Core Definition.** **API documentation best practices** are empirically established guidelines for producing documentation that is **accurate, discoverable, interactive, and maintainable**, derived from the OpenAPI ecosystem and developer-experience research.

**Technical Definition.** Best practices center on **one source of truth** (the OpenAPI document drives reference docs, SDKs, and mocks), **generation over hand-writing** (reference docs generated from the specification), **task-based navigation** (tags match product concepts, not code modules), **realistic examples** (not `"string"` placeholders), **complete error documentation** (`4xx` and `5xx` responses with bodies), **obvious authentication** (reachable in one click), **automated validation and linting in CI**, **automated publishing on every change**, **deliberate versioning with a changelog**, and **machine-readable output for AI agents**.

**Beginner-Friendly Explanation.** Best practices are **the habits that make documentation actually useful**. Keep one source of truth so docs and SDKs never disagree. Generate the reference instead of writing it by hand. Get developers to their first successful call fast. Show real examples they can copy. Document every error. Make authentication impossible to miss. Organize by what users want to do, not by how your code is structured. Let them try requests. Validate and publish automatically. Keep a changelog. These are the things that separate documentation developers love from documentation they ignore.

### Purposes

- **To reduce time-to-first-successful-call** for new integrators.
- **To eliminate drift** between documentation, SDKs, and the actual API behavior.
- **To reduce support volume** by answering questions before they become tickets.
- **To support multiple audiences** — evaluators, first-time integrators, maintainers, and AI agents.
- **To make documentation a reliable, automated part of the development lifecycle**.

### Syntax Rules and Structure

**Best-practice checklist.**

| Practice | Why It Matters | Quick Test |
|---|---|---|
| **One source of truth** | Docs, SDKs, and mocks cannot disagree | Is the reference generated from an OpenAPI document? |
| **Quickstart** | First success keeps people going | Can a new user make an authenticated call in under five minutes? |
| **Useful descriptions** | Names are not explanations | Does every operation have a summary that is not just the path? |
| **Real examples** | People copy, then adapt | Are there examples with realistic values, not `"string"`? |
| **Every error documented** | Errors are where people get stuck | Are `4xx` responses and their bodies described? |
| **Obvious authentication** | It is the first blocker | Is “how to get a key” reachable in one click from the home page? |
| **Task-based navigation** | Users search by goal | Do tags match product concepts, not code modules? |
| **Interactivity** | Trying beats reading | Can you send a request from the docs? |
| **Automated checks** | Humans forget | Is the OpenAPI document validated and linted in CI? |
| **Automated publishing** | Drift starts with manual steps | Do docs redeploy on every merge? |
| **Changelog** | Integrators need to plan | Is there a dated list of changes, with breaking ones called out? |
| **Machine-readable** | Agents and tools read docs too | Is the OpenAPI document downloadable, and is there an `llms.txt`? |

**Structure rules (Diátaxis framework).**

| Documentation type | Purpose |
|---|---|
| **Tutorial** | Learning-oriented; walks a newcomer through a first integration. |
| **How-to guide** | Task-oriented; solves a specific problem. |
| **Reference** | Information-oriented; describes endpoints, parameters, and schemas. |
| **Explanation** | Understanding-oriented; explains concepts and design decisions. |

**Constraints and limitations.**

- **Best practices are conventions**, not enforced by tooling; they require deliberate effort.
- **“One source of truth” requires tooling discipline**; a hand-edited reference alongside an OpenAPI document reintroduces drift.
- **AI-readable documentation is an emerging requirement**; plain-text formats and `llms.txt` are not yet universal.

### Annotated Complete Step by Step Code Examples

#### Example 1: A realistic OpenAPI operation with examples and errors

```yaml
paths:
  /users/{id}:
    get:
      summary: Retrieve a user by ID
      description: Returns the user's profile. Requires a valid bearer token.
      parameters:
        - name: id
          in: path
          required: true
          description: The numeric user identifier.
          schema:
            type: integer
            example: 42
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
              example:
                id: 42
                name: "Alice"
                email: "alice@example.com"
        '401':
          description: Missing or invalid token
          content:
            application/problem+json:
              schema:
                $ref: '#/components/schemas/Problem'
              example:
                type: "https://api.example.com/problems/unauthorized"
                title: "Unauthorized"
                status: 401
                detail: "The bearer token is missing or expired."
        '404':
          description: User not found
          content:
            application/problem+json:
              schema:
                $ref: '#/components/schemas/Problem'
              example:
                type: "https://api.example.com/problems/not-found"
                title: "User not found"
                status: 404
                detail: "No user exists with ID 999."
```

**Expected Output (conceptual):**
```
The operation has a meaningful summary, a real example value for the path parameter, a successful response example with realistic data, and documented 401 and 404 errors with problem+json examples.
```

**Why this output occurs.** Each best practice is applied: the `summary` explains the operation, the `example` fields use realistic values instead of placeholders, and every error response is documented with its schema and example body. This gives developers everything they need to succeed and to debug failures.

---

#### Example 2: CI validation and automated publishing

```yaml
# .github/workflows/docs.yml
name: Validate and Publish API Docs

on:
  push:
    branches: [main]
  pull_request:

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npx @redocly/cli lint openapi.yaml
      - run: npx swagger-cli validate openapi.yaml

  publish:
    needs: validate
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build:docs
      - run: npm run deploy:docs
```

**Expected Output (conceptual):**
```
On every pull request: OpenAPI document is linted and validated; the build fails on errors.
On every merge to main: documentation is rebuilt and deployed automatically.
```

**Why this output occurs.** The workflow enforces **automated validation and linting** on every change and **automated publishing** on merges. This eliminates manual steps and ensures documentation never drifts from the specification.

---

### Real-World Cases

- **Stripe, GitHub, and Twilio** generate reference documentation from OpenAPI descriptions and enforce validation in CI.
- **Mintlify and Scalar** provide documentation platforms that generate interactive references directly from OpenAPI documents.
- **AI-agent-ready documentation** is an emerging practice: publishing downloadable OpenAPI documents and plain-text versions of documentation pages so agents can consume them.

---

## References

- OpenAPI Specification v3.1.1 — OpenAPI Initiative - https://spec.openapis.org/oas/v3.1.1.html
- Swagger UI — Installation - https://swagger.io/docs/open-source-tools/swagger-ui/usage/installation/
- Swagger UI — Configuration - https://swagger.io/docs/open-source-tools/swagger-ui/usage/configuration/
- Swagger UI — GitHub Repository - https://github.com/swagger-api/swagger-ui
- swagger-jsdoc — npm - https://www.npmjs.com/package/swagger-jsdoc
- swagger-jsdoc — Getting Started (README) - https://github.com/Surnet/swagger-jsdoc
- swagger-ui-express — npm - https://www.npmjs.com/package/swagger-ui-express
- swagger-ui-express — GitHub Repository - https://github.com/scottie1984/swagger-ui-express
- API Documentation Guide — Mintlify - https://www.mintlify.com/library/api-documentation-guide
- API Documentation Best Practices — Scalar - https://github.com/scalar/scalar/blob/main/documentation/learn/openapi/api-documentation-best-practices.md
- OpenAPI vs. Swagger — Microsoft Learn (ASP.NET Core) - https://learn.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger
- OpenAPI Specification — Wikipedia - https://en.wikipedia.org/wiki/OpenAPI_Specification
- A Brief History of OpenAPI — Bump.sh - https://docs.bump.sh/guides/openapi/history/
- RFC 9457: Problem Details for HTTP APIs — IETF - https://datatracker.ietf.org/doc/html/rfc9457