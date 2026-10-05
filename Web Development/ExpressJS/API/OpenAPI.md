# OpenAPI (Swagger) Specification & Fundamentals — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** The OpenAPI Specification (OAS) is a vendor-neutral, language-agnostic standard for describing HTTP APIs in a machine-readable format (JSON or YAML). It defines a structured contract that describes an API's paths, operations, parameters, request bodies, responses, schemas, and security requirements. 

**Technical Definition:** The OpenAPI Specification defines a standard, programming language-agnostic interface description for HTTP APIs, allowing both humans and computers to discover and understand the capabilities of a service without requiring access to source code, additional documentation, or network traffic inspection. An OpenAPI document is a JSON or YAML file that conforms to the specification, using a defined schema to describe API endpoints, operations, parameters, and data models. The specification is maintained by the OpenAPI Initiative under the Linux Foundation. 

**Beginner-Friendly Explanation:** An OpenAPI document is like a blueprint for your API. Instead of writing documentation by hand that gets out of date, you write a single file that describes every endpoint, what parameters it takes, what data it returns, and how it's secured. Tools can then read this file to generate interactive documentation (like Swagger UI), client SDKs, server stubs, mock servers, and tests — all from the same source of truth.

### Key Characteristics

- **Language-agnostic:** Describes APIs in JSON or YAML, independent of programming language or framework.
- **Machine-readable:** Enables automated tooling for documentation, code generation, testing, and mocking.
- **Contract-first:** Can be written before implementation, serving as a contract between teams.
- **Evolving standard:** OpenAPI 3.1 aligns with JSON Schema 2020-12; OpenAPI 3.2 adds QUERY method and webhooks enhancements.
- **Swagger is the tooling:** Swagger UI, Swagger Editor, and Swagger Codegen are tools that work with OpenAPI documents.
- **Component reuse:** Reusable schemas, parameters, responses, and security schemes reduce duplication.

### Prerequisites

- **Basic understanding of HTTP:** Methods, status codes, headers, and the request–response cycle.
- **Familiarity with REST API concepts:** Resources, endpoints, and CRUD operations.
- **Basic knowledge of JSON or YAML syntax.**
- **Understanding of JSON Schema concepts:** types, properties, required, etc.

### Related Programming Areas

- **API documentation:** Swagger UI, Redoc, and Mintlify render OpenAPI documents as interactive docs.
- **Code generation:** OpenAPI Generator and Swagger Codegen generate client SDKs and server stubs.
- **Contract testing:** Tools like Dredd and Schemathesis validate implementations against the spec.
- **API gateways:** AWS API Gateway, Kong, and Azure API Management import OpenAPI documents.
- **Mock servers:** Prism and Stoplight generate mock APIs from OpenAPI documents.

### Core Concepts

1. **OpenAPI 3.0/3.1 vs. Swagger 2.0** — version differences and migration considerations.
2. **Paths, HTTP Methods, and Operations** — the structural backbone of an API description.
3. **Parameters** — path, query, header, and cookie parameters.
4. **Request Bodies and Media Types** — describing request payloads.
5. **Data Schemas** — reusable components using `components/schemas`.
6. **HTTP Responses and Status Codes** — describing API responses.
7. **Security Schemes** — Bearer tokens, OAuth2, API keys, and mutual TLS.

---

## Core Concept 1: OpenAPI 3.0/3.1 Standard vs. Swagger 2.0

### Definitions

**Core Definition:** Swagger 2.0 and OpenAPI 2.0 are the same specification; Swagger 2.0 was donated to the OpenAPI Initiative in 2015 and renamed OpenAPI 2.0. OpenAPI 3.0 and 3.1 are subsequent versions with significant structural and capability improvements. 

**Technical Definition:** In Swagger 2.0, a property called `swagger` indicated the specification version. In OpenAPI 3.0, this was replaced by the `openapi` property. Swagger 2.0 uses a defined subset of JSON Schema Draft 4, while OpenAPI 3.0 uses an OpenAPI-specific schema model, and OpenAPI 3.1 aligns its Schema Object with JSON Schema Draft 2020-12. OpenAPI 3.1 introduces webhooks, full JSON Schema compatibility, `$ref` with sibling keywords, and support for `if`/`then`/`else`, `const`, and `prefixItems`. 

**Beginner-Friendly Explanation:** Swagger 2.0 was the original name for the specification. In 2015, the company that created it donated it to an open foundation, and it was renamed OpenAPI 2.0. Since then, newer versions (3.0 and 3.1) have added features like the ability to describe webhooks, support for modern JSON Schema validation rules, and better ways to reuse components. If you're starting a new project, OpenAPI 3.1 is recommended.

### Purposes

- To understand the evolution of the API description standard and choose the right version for your project.
- To plan migration from legacy Swagger 2.0 documents to modern OpenAPI versions.
- To leverage newer features like webhooks, JSON Schema 2020-12, and `if`/`then`/`else` validation.
- To ensure compatibility with tooling that may support different versions.

### Syntax Rules and Structure

| Feature | Swagger 2.0 | OpenAPI 3.0 | OpenAPI 3.1 |
|---------|-------------|-------------|-------------|
| Version indicator | `swagger: "2.0"` | `openapi: 3.0.x` | `openapi: 3.1.0` |
| JSON Schema | Subset of Draft 4 | OpenAPI-specific model | Full 2020-12 |
| `$ref` with siblings | ❌ | ❌ (needs `allOf`) | ✅ |
| `nullable` (keyword) | ⚠️ (`x-nullable`) | ✅ | ⚠️ (deprecated) |
| `nullable` (type array) | ❌ | ❌ | ✅ |
| `oneOf` / `anyOf` | ❌ | ✅ | ✅ |
| `const` | ❌ | ❌ | ✅ |
| `if`/`then`/`else` | ❌ | ❌ | ✅ |
| `prefixItems` (tuples) | ❌ | ❌ | ✅ |
| Webhooks | ❌ | ❌ | ✅ |
| Multiple content types | ❌ | ✅ | ✅ |
| Server URLs | `host` + `basePath` | `servers` array | `servers` array |
| Security schemes | `securityDefinitions` | `components/securitySchemes` | `components/securitySchemes` |
| Mutual TLS | ❌ | ❌ | ✅ |

**Rules:**
- Use `swagger: "2.0"` for Swagger 2.0 documents; use `openapi: 3.0.x` or `openapi: 3.1.0` for OpenAPI 3.x documents. 
- OpenAPI 3.1 is recommended for new projects due to its wider support base and alignment with modern JSON Schema. 
- Swagger 2.0 documents can be migrated directly to OpenAPI 3.1 or via OpenAPI 3.0. 
- OpenAPI 3.1 allows a valid document to contain only `webhooks` or only `components`, without `paths`. 

### Annotated Code Example

```yaml
# openapi-3.1-example.yaml
openapi: 3.1.0
info:
  title: Example API
  version: 1.0.0
  summary: A sample API demonstrating OpenAPI 3.1 features

# Webhooks (new in OpenAPI 3.1)
webhooks:
  newPet:
    post:
      summary: New pet notification
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Pet'
      responses:
        '200':
          description: Webhook received

# Paths (required in Swagger 2.0, optional in 3.1)
paths:
  /pets:
    get:
      summary: List all pets
      responses:
        '200':
          description: A list of pets
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Pet'

components:
  schemas:
    Pet:
      type: object
      required: [id, name]
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
        tag:
          type: [string, "null"]  # JSON Schema 2020-12 type array
```

**Expected Output (rendered by Swagger UI):**
```
Interactive documentation showing:
- Webhook: newPet (POST)
- Path: GET /pets — List all pets
- Schema: Pet (id, name, tag with nullable type)
```

**Why this output:** The OpenAPI 3.1 document uses `type: [string, "null"]` (a JSON Schema 2020-12 feature) to indicate a nullable field, and includes a `webhooks` section (new in 3.1). Swagger UI renders the webhook, the path, and the schema interactively. In Swagger 2.0, the webhook would not be expressible, and nullable would require `x-nullable`.

### Real-World Cases

- **New projects:** Start with OpenAPI 3.1 for full JSON Schema compatibility and webhook support.
- **Legacy migration:** Swagger 2.0 → OpenAPI 3.0 → OpenAPI 3.1 for a gradual upgrade path.
- **Tooling compatibility:** Some older tools only support Swagger 2.0 or OpenAPI 3.0; check tool support before choosing a version.

---

## Core Concept 2: Paths, HTTP Methods, and Operations

### Definitions

**Core Definition:** Paths are the URL endpoints of an API (e.g., `/users/{id}`), and operations are the HTTP methods (GET, POST, PUT, DELETE, etc.) that can be performed on those paths.

**Technical Definition:** The `paths` object holds the relative paths to individual endpoints. Each path item defines one or more operations, using HTTP methods as keys (`get`, `post`, `put`, `patch`, `delete`, `head`, `options`, `trace`). Each operation is an Operation Object containing parameters, request body, responses, and metadata (summary, description, operationId, tags). Path templating uses curly braces (`{}`) to denote path parameters. 

**Beginner-Friendly Explanation:** A path is like a street address (`/users/42`), and an operation is like an action you can perform at that address (GET to read, DELETE to remove). OpenAPI lets you describe every possible action at every address, including what parameters each action takes and what it returns.

### Purposes

- To define the structural backbone of an API description — every endpoint and the operations available on it.
- To organise operations logically using paths and path parameters.
- To provide human-readable summaries and descriptions for each operation.
- To assign unique `operationId` values for code generation and tooling.

### Syntax Rules and Structure

```yaml
paths:
  /users:
    get:
      summary: List users
      operationId: listUsers
      tags: [Users]
      parameters:
        - name: limit
          in: query
          schema: { type: integer }
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
      summary: Create user
      operationId: createUser
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/User'
      responses:
        '201':
          description: User created
```

| HTTP Method | Purpose | Typical Status Codes |
|-------------|---------|---------------------|
| `get` | Retrieve a resource or collection | 200, 404 |
| `post` | Create a resource | 201, 400, 409 |
| `put` | Replace a resource | 200, 404 |
| `patch` | Partially update a resource | 200, 404 |
| `delete` | Remove a resource | 204, 404 |
| `head` | Retrieve headers only | 200 |
| `options` | Get allowed methods | 200, 204 |
| `trace` | Diagnostic trace | 200 |

**Rules:**
- Method names must be lowercase in OpenAPI documents. 
- `operationId` must be unique across the entire document; it is used for code generation.
- `tags` group operations in documentation UIs (e.g., Swagger UI, Redoc).
- Path parameters must be defined with `required: true` and `in: path`. 
- OpenAPI 3.2 adds the `QUERY` method and `additionalOperations` for custom methods. 

### Annotated Code Example

```yaml
# paths-operations.yaml
openapi: 3.1.0
info:
  title: Blog API
  version: 1.0.0

paths:
  /posts:
    get:
      summary: List blog posts
      operationId: listPosts
      tags: [Posts]
      parameters:
        - name: page
          in: query
          schema: { type: integer, default: 1 }
        - name: limit
          in: query
          schema: { type: integer, default: 20, maximum: 100 }
      responses:
        '200':
          description: Paginated list of posts
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/Post'
                  meta:
                    $ref: '#/components/schemas/PaginationMeta'
    post:
      summary: Create a blog post
      operationId: createPost
      tags: [Posts]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PostInput'
      responses:
        '201':
          description: Post created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Post'
        '400':
          description: Validation error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  /posts/{postId}:
    parameters:
      - name: postId
        in: path
        required: true
        schema: { type: string, format: uuid }
    get:
      summary: Get a single post
      operationId: getPost
      tags: [Posts]
      responses:
        '200':
          description: Post details
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Post'
        '404':
          description: Post not found
    delete:
      summary: Delete a post
      operationId: deletePost
      tags: [Posts]
      responses:
        '204':
          description: Post deleted
```

**Expected Output (rendered by Swagger UI):**
```
Posts
  GET /posts — List blog posts
  POST /posts — Create a blog post
  GET /posts/{postId} — Get a single post
  DELETE /posts/{postId} — Delete a post
```

**Why this output:** The `paths` object defines two path items: `/posts` and `/posts/{postId}`. The first has `get` and `post` operations; the second has a shared `parameters` array (path parameter `postId`) and `get` and `delete` operations. Swagger UI groups all operations tagged "Posts" under a single section.

### Real-World Cases

- **REST APIs:** Every resource is a path with CRUD operations as HTTP methods.
- **Versioned APIs:** Paths include version prefixes (`/api/v1/users`).
- **Nested resources:** Paths like `/users/{userId}/orders/{orderId}` model hierarchical relationships.
- **Webhook paths:** OpenAPI 3.1 allows describing webhook paths in a separate `webhooks` object.

---

## Core Concept 3: Parameters (Path, Query, Header, Cookie)

### Definitions

**Core Definition:** Parameters are inputs to an operation that are not part of the request body — they are passed in the URL path, query string, headers, or cookies.

**Technical Definition:** OpenAPI 3.0 distinguishes four parameter locations based on the `in` field: `path` (part of the URL path, always required), `query` (appended to the URL), `header` (custom request headers), and `cookie` (passed in the `Cookie` header). Each parameter has a `name`, `in`, optional `description`, optional `required` (defaults to `false` for query/header/cookie, must be `true` for path), and a `schema` defining its data type. Parameters can be serialised using `style` and `explode` keywords for arrays and objects. 

**Beginner-Friendly Explanation:** Parameters are the little pieces of information you send along with a request — like the user ID in `/users/42` (path parameter), the page number in `?page=2` (query parameter), an API key in a header (header parameter), or a session token in a cookie (cookie parameter). OpenAPI lets you describe all of them with their types, whether they're required, and how they should be formatted.

### Purposes

- To describe all non-body inputs to an operation, including their types, formats, and validation rules.
- To enable client code generators to create correctly typed function signatures.
- To allow documentation tools to display parameter descriptions and constraints.
- To define serialisation rules for arrays and objects in query strings and paths.

### Syntax Rules and Structure

```yaml
parameters:
  # Path parameter (always required)
  - name: userId
    in: path
    required: true
    description: The user's unique identifier
    schema:
      type: string
      format: uuid

  # Query parameter (optional by default)
  - name: status
    in: query
    description: Filter by user status
    required: false
    schema:
      type: string
      enum: [active, inactive, pending]
      default: active

  # Query parameter with array (form style, exploded)
  - name: tags
    in: query
    style: form
    explode: true
    schema:
      type: array
      items:
        type: string

  # Header parameter
  - name: X-Request-Id
    in: header
    description: Unique request identifier for tracing
    schema:
      type: string
      format: uuid

  # Cookie parameter
  - name: session_token
    in: cookie
    schema:
      type: string
```

| Parameter Location | `in` Value | Required Default | Example |
|-------------------|-----------|-----------------|---------|
| Path | `path` | `true` (must be) | `/users/{id}` |
| Query | `query` | `false` | `/users?role=admin` |
| Header | `header` | `false` | `X-My-Header: Value` |
| Cookie | `cookie` | `false` | `Cookie: debug=0` |

**Serialisation Styles:**

| Style | Location | Array Example | Object Example |
|-------|----------|--------------|----------------|
| `form` (default for query) | Query/Cookie | `?color=blue,green` | `?R=100&G=200` |
| `spaceDelimited` | Query | `?color=blue%20green` | — |
| `pipeDelimited` | Query | `?color=blue|green` | — |
| `deepObject` | Query | — | `?color[R]=100&color[G]=200` |
| `simple` (default for path/header) | Path/Header | `/users/12,34,56` | `/users/R,100,G,200` |

**Rules:**
- Path parameters must have `required: true`. 
- The parameter `name` for path parameters must match the template variable name (e.g., `{id}` → `name: id`). 
- `explode: true` (the default for `style: form`) generates separate parameters for each array item (`?color=blue&color=green`); `explode: false` generates a comma-delimited value (`?color=blue,green`). 
- To describe API keys passed as query parameters, use `securitySchemes` and `security` instead of a parameter definition. 
- Parameters can be defined at the path level (shared by all operations) or at the operation level.

### Annotated Code Example

```yaml
# parameters.yaml
openapi: 3.1.0
info:
  title: Product API
  version: 1.0.0

paths:
  /products:
    get:
      summary: Search products
      parameters:
        - name: q
          in: query
          description: Search term
          schema: { type: string, minLength: 1, maxLength: 100 }
        - name: category
          in: query
          schema:
            type: string
            enum: [electronics, furniture, clothing]
        - name: minPrice
          in: query
          schema: { type: number, format: float, minimum: 0 }
        - name: tags
          in: query
          style: form
          explode: true
          schema:
            type: array
            items: { type: string }
        - name: X-API-Version
          in: header
          schema: { type: string, enum: ['1', '2'], default: '2' }
        - name: session
          in: cookie
          schema: { type: string }
      responses:
        '200':
          description: Search results

  /products/{productId}:
    parameters:
      - name: productId
        in: path
        required: true
        schema: { type: string, format: uuid }
    get:
      summary: Get product by ID
      responses:
        '200':
          description: Product details
```

**Expected Output (rendered parameter table in Swagger UI):**
```
Query Parameters:
  q          string     Search term
  category   string     (electronics, furniture, clothing)
  minPrice   number     >= 0
  tags       array      (exploded: ?tags=a&tags=b)

Header Parameters:
  X-API-Version  string  (1, 2, default: 2)

Cookie Parameters:
  session  string

Path Parameters:
  productId  string (uuid)  required
```

**Why this output:** The query parameters include a string with length constraints, an enum, a number with a minimum, and an exploded array (`?tags=a&tags=b`). The header parameter is an enum with a default. The cookie parameter is a simple string. The path parameter is marked required and uses UUID format. Swagger UI renders each with its constraints.

### Real-World Cases

- **Filtering and pagination:** Query parameters for `?page=2&limit=20&sort=createdAt`.
- **Resource identification:** Path parameters like `/users/{userId}`.
- **API versioning:** Header parameter `X-API-Version` to select API version.
- **Session management:** Cookie parameters for session tokens or CSRF tokens.
- **Tracing:** Header parameter `X-Request-Id` for distributed tracing.

---

## Core Concept 4: Request Bodies and Media Types

### Definitions

**Core Definition:** A request body is the payload sent with POST, PUT, or PATCH operations, described in OpenAPI using the `requestBody` keyword and a `content` map that associates media types with schemas.

**Technical Definition:** In OpenAPI 3.0, the `requestBody` keyword replaces the `body` and `formData` parameters from Swagger 2.0. The `requestBody` consists of an optional `description`, an optional `required` flag (default `false`), and a required `content` object. The `content` map's keys are media types (e.g., `application/json`, `application/xml`, `multipart/form-data`) and values are Media Type Objects, each with a `schema`, optional `example` or `examples`, and optional `encoding` for form data. GET, DELETE, and HEAD operations are not allowed to have request bodies. 

**Beginner-Friendly Explanation:** When you send data to an API — like creating a new user or updating a product — that data goes in the request body. OpenAPI lets you describe exactly what format that data should be in (JSON, XML, form data) and what fields it should contain. You can even describe multiple formats: the API might accept both JSON and XML.

### Purposes

- To describe the structure and format of data that clients send to the API.
- To enable client code generators to create correctly typed request payloads.
- To support multiple media types (JSON, XML, form data, plain text) with different schemas.
- To provide examples for documentation and testing.
- To specify form data encoding for file uploads and complex form structures.

### Syntax Rules and Structure

```yaml
requestBody:
  description: User to create
  required: true
  content:
    application/json:
      schema:
        $ref: '#/components/schemas/UserInput'
      examples:
        standard:
          summary: A standard user
          value:
            name: Alice
            email: alice@example.com
    application/xml:
      schema:
        $ref: '#/components/schemas/UserInput'
    multipart/form-data:
      schema:
        type: object
        properties:
          avatar:
            type: string
            format: binary
          name:
            type: string
```

| Media Type | Use Case | Schema Type |
|-----------|----------|-------------|
| `application/json` | Most REST APIs | Object, array |
| `application/xml` | Legacy/enterprise systems | Object |
| `multipart/form-data` | File uploads, mixed fields | Object with `format: binary` |
| `application/x-www-form-urlencoded` | HTML form submissions | Object |
| `text/plain` | Simple text payloads | String |
| `image/*` | Binary image uploads | String (`format: binary`) |

**Rules:**
- `requestBody` is optional by default; use `required: true` to mark it mandatory. 
- `content` allows wildcard media types (`image/*`, `*/*`); specific types take precedence (`image/png` > `image/*` > `*/*`). 
- GET, DELETE, and HEAD operations cannot have request bodies. 
- `anyOf` and `oneOf` can be used to specify alternate schemas for the request body. 
- Request body definitions can be reused via `components/requestBodies` and `$ref`. 

### Annotated Code Example

```yaml
# request-body.yaml
openapi: 3.1.0
info:
  title: User API
  version: 1.0.0

paths:
  /users:
    post:
      summary: Create a user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserInput'
            examples:
              minimal:
                summary: Minimal user
                value:
                  name: Alice
              complete:
                summary: Complete user
                value:
                  name: Bob
                  email: bob@example.com
                  role: admin
          multipart/form-data:
            schema:
              type: object
              required: [name]
              properties:
                name:
                  type: string
                avatar:
                  type: string
                  format: binary
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'

components:
  schemas:
    UserInput:
      type: object
      required: [name]
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 100
        email:
          type: string
          format: email
        role:
          type: string
          enum: [admin, user, viewer]
          default: user
    User:
      allOf:
        - $ref: '#/components/schemas/UserInput'
        - type: object
          properties:
            id:
              type: string
              format: uuid
            createdAt:
              type: string
              format: date-time
```

**Expected Output (rendered in Swagger UI):**
```
POST /users
  Request Body (required):
    application/json:
      Schema: UserInput
      Examples:
        minimal: {"name": "Alice"}
        complete: {"name": "Bob", "email": "bob@example.com", "role": "admin"}
    multipart/form-data:
      Schema: { name (string, required), avatar (binary) }
  Responses:
    201: User created
```

**Why this output:** The `requestBody` describes two media types: JSON (with the `UserInput` schema and two examples) and multipart form data (for file uploads). The `UserInput` schema defines required fields, constraints, and defaults. The `User` schema uses `allOf` to combine `UserInput` with server-generated fields (`id`, `createdAt`). Swagger UI renders the request body with schema, examples, and media type selection.

### Real-World Cases

- **User registration:** `POST /users` with a JSON body containing name, email, and password.
- **File uploads:** `POST /upload` with `multipart/form-data` for file and metadata fields.
- **Bulk operations:** `POST /users/bulk` with an array of user objects.
- **Content negotiation:** Supporting both JSON and XML for the same endpoint.

---

## Core Concept 5: Data Schemas (Reusable Components)

### Definitions

**Core Definition:** Data schemas define the structure of request and response payloads; reusable components in `components/schemas` allow schemas to be defined once and referenced from multiple places using `$ref`.

**Technical Definition:** The Components Object, accessible through the `components` field in the root OpenAPI Object, contains definitions for objects to be reused in other parts of the description. Most objects in an OpenAPI document can be replaced by a reference to a component using a JSON Reference (`$ref`). The `components/schemas` map holds Schema Objects that describe data structures using JSON Schema keywords (`type`, `properties`, `required`, `enum`, `format`, `minimum`, `maxLength`, etc.). In OpenAPI 3.1, the Schema Object is fully compatible with JSON Schema Draft 2020-12, enabling `const`, `if`/`then`/`else`, `prefixItems`, and `$ref` with sibling keywords. 

**Beginner-Friendly Explanation:** Instead of repeating the same data structure definition every time you describe a request or response, you define it once in `components/schemas` and reference it with `$ref`. This is like defining a class in programming — you write it once and use it everywhere. If you need to change the structure, you change it in one place and all references update automatically.

### Purposes

- To define reusable data structures that can be referenced from request bodies, response bodies, and parameters.
- To reduce duplication and maintenance burden in large OpenAPI documents.
- To provide a single source of truth for data models shared between teams and services.
- To enable code generators to produce consistent model classes.
- To leverage advanced JSON Schema features (`oneOf`, `anyOf`, `allOf`, `if`/`then`/`else`) for complex validation.

### Syntax Rules and Structure

```yaml
components:
  schemas:
    User:
      type: object
      required: [id, name, email]
      properties:
        id:
          type: string
          format: uuid
          readOnly: true
        name:
          type: string
          minLength: 1
          maxLength: 100
        email:
          type: string
          format: email
        role:
          type: string
          enum: [admin, user, viewer]
          default: user
        createdAt:
          type: string
          format: date-time
          readOnly: true

    Error:
      type: object
      required: [code, message]
      properties:
        code:
          type: string
          example: VALIDATION_ERROR
        message:
          type: string
        details:
          type: array
          items:
            type: object
            properties:
              field: { type: string }
              message: { type: string }

    PaginationMeta:
      type: object
      properties:
        page: { type: integer, minimum: 1 }
        limit: { type: integer, minimum: 1 }
        total: { type: integer, minimum: 0 }
```

| JSON Schema Keyword | Purpose | Example |
|--------------------|---------|---------|
| `type` | Data type | `string`, `integer`, `array`, `object` |
| `format` | Semantic format | `uuid`, `email`, `date-time` |
| `enum` | Allowed values | `[admin, user, viewer]` |
| `required` | Required properties | `[id, name, email]` |
| `readOnly` | Server-generated only | `true` for `id`, `createdAt` |
| `allOf` | Combine schemas | Merge base + extension |
| `oneOf` | Exactly one schema matches | Polymorphic types |
| `anyOf` | At least one schema matches | Flexible types |
| `const` (3.1) | Constant value | `const: "api-v1"` |
| `if`/`then`/`else` (3.1) | Conditional validation | Conditional required fields |
| `prefixItems` (3.1) | Tuple validation | `[string, integer]` |

**Rules:**
- Component names must match the pattern `^[a-zA-Z0-9\.\-_]+$`. 
- `$ref` values are URI references; they can be internal (`#/components/schemas/User`) or external (`./shared.yaml#/components/schemas/User`). 
- In OpenAPI 3.1, `$ref` can have sibling keywords (e.g., `description` alongside `$ref`), unlike OpenAPI 3.0 where `allOf` was required. 
- `readOnly: true` marks properties that are returned in responses but ignored in requests. 
- `writeOnly: true` marks properties that are accepted in requests but never returned in responses. 

### Annotated Code Example

```yaml
# components-schemas.yaml
openapi: 3.1.0
info:
  title: E-Commerce API
  version: 1.0.0

paths:
  /products:
    get:
      summary: List products
      responses:
        '200':
          description: Product list
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/Product'
                  meta:
                    $ref: '#/components/schemas/PaginationMeta'
    post:
      summary: Create product
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ProductInput'
      responses:
        '201':
          description: Product created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Product'
        '422':
          description: Validation error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

components:
  schemas:
    ProductBase:
      type: object
      required: [name, price]
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 200
        price:
          type: number
          format: float
          minimum: 0
        category:
          type: string
          enum: [electronics, furniture, clothing]
        tags:
          type: array
          items: { type: string }
          maxItems: 10

    ProductInput:
      allOf:
        - $ref: '#/components/schemas/ProductBase'

    Product:
      allOf:
        - $ref: '#/components/schemas/ProductBase'
        - type: object
          properties:
            id:
              type: string
              format: uuid
              readOnly: true
            createdAt:
              type: string
              format: date-time
              readOnly: true
            updatedAt:
              type: string
              format: date-time
              readOnly: true

    Error:
      type: object
      required: [code, message]
      properties:
        code: { type: string }
        message: { type: string }
        details:
          type: array
          items:
            type: object
            properties:
              field: { type: string }
              message: { type: string }

    PaginationMeta:
      type: object
      properties:
        page: { type: integer, minimum: 1 }
        limit: { type: integer, minimum: 1 }
        total: { type: integer, minimum: 0 }
```

**Expected Output (rendered in Swagger UI):**
```
Schemas:
  ProductBase: { name, price, category, tags }
  ProductInput: (allOf → ProductBase)
  Product: (allOf → ProductBase + id, createdAt, updatedAt)
  Error: { code, message, details }
  PaginationMeta: { page, limit, total }

Paths:
  GET /products → Product[]
  POST /products → ProductInput → Product | Error
```

**Why this output:** `ProductBase` defines shared fields. `ProductInput` uses `allOf` to reference `ProductBase` (no additional fields — used for creating). `Product` uses `allOf` to combine `ProductBase` with server-generated fields (`id`, `createdAt`, `updatedAt` marked `readOnly`). `Error` and `PaginationMeta` are standalone reusable schemas. Swagger UI renders each schema with its properties and relationships.

### Real-World Cases

- **Shared models across microservices:** A `User` schema referenced by multiple services.
- **Polymorphic types:** Using `oneOf` to describe different payment method types (`CreditCard`, `PayPal`, `BankTransfer`).
- **Validation rules:** Using `if`/`then`/`else` (OpenAPI 3.1) to conditionally require fields.
- **Read-only vs. write-only fields:** `readOnly` for server-generated IDs; `writeOnly` for passwords.
- **Pagination metadata:** A reusable `PaginationMeta` schema for all list endpoints.

---

## Core Concept 6: HTTP Responses and Status Codes

### Definitions

**Core Definition:** The `responses` object describes the possible responses from an operation, keyed by HTTP status code (e.g., `200`, `404`), with each response containing a description, optional headers, and optional content (body schema).

**Technical Definition:** The Responses Object is a container for the expected responses of an operation. It MUST contain at least one response code, and SHOULD contain the response for a successful operation call. Keys are HTTP status codes (e.g., `"200"`), status code ranges using the wildcard `X` (e.g., `"2XX"` for all 2xx codes), or `default` for responses not covered by specific codes. Each response is a Response Object with a required `description`, optional `headers` map, and optional `content` map. The `content` map associates media types with schemas, following the same structure as `requestBody.content`. 

**Beginner-Friendly Explanation:** When someone calls your API, they need to know what to expect back. The `responses` object tells them: "If everything goes well, you'll get a 200 with this JSON structure. If the resource doesn't exist, you'll get a 404 with this error structure. If there's a validation error, you'll get a 422 with field-level details." This lets client developers handle every possible outcome.

### Purposes

- To describe every possible outcome of an operation, including success and error cases.
- To provide response body schemas for code generation and documentation.
- To document response headers (e.g., pagination links, rate limit headers).
- To group status code ranges with the wildcard `X` syntax.
- To ensure clients know how to handle errors, not just successful responses.

### Syntax Rules and Structure

```yaml
responses:
  '200':
    description: Successful response
    headers:
      X-RateLimit-Limit:
        schema: { type: integer }
      X-RateLimit-Remaining:
        schema: { type: integer }
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/User'
  '201':
    description: Resource created
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/User'
  '400':
    description: Bad request
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/Error'
  '401':
    description: Unauthorized
  '404':
    description: Resource not found
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/Error'
  '422':
    description: Validation error
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/Error'
  '2XX':
    description: Any successful response
  default:
    description: Unexpected error
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/Error'
```

| Status Code | Common Use | Response Body |
|-------------|-----------|---------------|
| `200` | Successful GET, PUT, PATCH | Resource or collection |
| `201` | Successful POST (created) | New resource |
| `204` | Successful DELETE, PUT (no body) | No content |
| `400` | Malformed request | Error details |
| `401` | Missing/invalid auth | Error or challenge |
| `403` | Insufficient permissions | Error |
| `404` | Resource not found | Error |
| `409` | Conflict (duplicate, concurrency) | Error |
| `422` | Semantic validation error | Field-level errors |
| `500` | Server error | Generic error |

**Rules:**
- The `responses` object MUST contain at least one response code. 
- Status codes must be enclosed in quotation marks (e.g., `"200"`) for JSON/YAML compatibility. 
- The wildcard `X` can be used to define ranges: `1XX`, `2XX`, `3XX`, `4XX`, `5XX`. An explicit code definition takes precedence over a range definition. 
- The `default` response covers all status codes not explicitly defined. 
- The `description` field is REQUIRED for every response. 
- Response headers are defined in the `headers` map, with header names as keys. 
- `Content-Type` headers are ignored in the `headers` map (use `content` instead). 

### Annotated Code Example

```yaml
# responses.yaml
openapi: 3.1.0
info:
  title: User API
  version: 1.0.0

paths:
  /users/{userId}:
    get:
      summary: Get user by ID
      parameters:
        - name: userId
          in: path
          required: true
          schema: { type: string, format: uuid }
      responses:
        '200':
          description: User found
          headers:
            X-Request-Id:
              description: Unique request identifier
              schema: { type: string, format: uuid }
            Cache-Control:
              schema: { type: string, example: 'max-age=3600' }
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '401':
          description: Authentication required
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        '403':
          description: Insufficient permissions
        '404':
          description: User not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
              example:
                code: USER_NOT_FOUND
                message: No user with the specified ID exists
        '4XX':
          description: Client error
        '5XX':
          description: Server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
        default:
          description: Unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

components:
  schemas:
    User:
      type: object
      properties:
        id: { type: string, format: uuid }
        name: { type: string }
        email: { type: string, format: email }
    Error:
      type: object
      required: [code, message]
      properties:
        code: { type: string }
        message: { type: string }
```

**Expected Output (rendered in Swagger UI):**
```
GET /users/{userId}
  Responses:
    200: User found
      Headers: X-Request-Id, Cache-Control
      Body: User
    401: Authentication required — Error
    403: Insufficient permissions
    404: User not found — Error (example: USER_NOT_FOUND)
    4XX: Client error
    5XX: Server error — Error
    default: Unexpected error — Error
```

**Why this output:** The `responses` object defines specific codes (200, 401, 403, 404), ranges (4XX, 5XX), and a `default` fallback. The 200 response includes custom headers (`X-Request-Id`, `Cache-Control`) and the `User` schema. The 404 response includes an inline example showing the error structure. Swagger UI renders all responses with their descriptions, headers, and schemas.

### Real-World Cases

- **REST APIs:** Every operation should define at least a 200/201 success response and common error responses (400, 401, 404, 500).
- **API gateways:** Response definitions drive validation and mocking.
- **Client SDKs:** Response schemas generate return types for client methods.
- **Monitoring:** Documenting error responses helps clients implement retry and fallback logic.

---

## Core Concept 7: Security Schemes (Bearer Tokens, OAuth2, API Keys)

### Definitions

**Core Definition:** Security schemes define how an API is protected — using HTTP authentication (Bearer/Basic), API keys, OAuth 2.0, OpenID Connect, or mutual TLS — and are declared in `components/securitySchemes` and referenced in operations or globally.

**Technical Definition:** OpenAPI 3.1 supports five security scheme types: **HTTP authentication** (using the `Authorization` header, with schemes like `bearer` and `basic`), **API keys** (in headers, query parameters, or cookies), **OAuth 2.0** (with flows: authorizationCode, implicit, password, clientCredentials), **OpenID Connect** (with a discovery URL), and **mutual TLS** (client certificate authentication). Security schemes are defined in `components.securitySchemes` and applied via the `security` field at the root level (global) or operation level (override). An empty security array (`security: []`) makes security optional for a specific operation. 

**Beginner-Friendly Explanation:** Security schemes tell clients how to authenticate with your API. A Bearer token scheme means "send your token in the `Authorization: Bearer <token>` header." An API key scheme means "send your key in this header or query parameter." OAuth 2.0 means "go through this authorization flow to get a token." OpenAPI lets you describe all of these so client developers know exactly how to authenticate.

### Purposes

- To document how clients authenticate with the API.
- To enable Swagger UI to provide an "Authorize" button for testing authenticated endpoints.
- To allow code generators to create authentication boilerplate.
- To define OAuth 2.0 flows with authorization and token URLs.
- To support multiple security schemes and combinations (AND/OR logic).

### Syntax Rules and Structure

```yaml
components:
  securitySchemes:
    # Bearer token (JWT)
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT access token from the auth server

    # API key in header
    ApiKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key
      description: API key for service-to-service calls

    # API key in query parameter
    ApiKeyQuery:
      type: apiKey
      in: query
      name: api_key

    # API key in cookie
    ApiKeyCookie:
      type: apiKey
      in: cookie
      name: session_token

    # OAuth 2.0 with authorization code flow
    OAuth2:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://auth.example.com/authorize
          tokenUrl: https://auth.example.com/token
          refreshUrl: https://auth.example.com/refresh
          scopes:
            read: Read access
            write: Write access
            admin: Admin access

    # OpenID Connect
    OpenID:
      type: openIdConnect
      openIdConnectUrl: https://auth.example.com/.well-known/openid-configuration

    # Mutual TLS
    MutualTLS:
      type: mutualTLS
      description: Client certificate required

# Global security (applies to all operations unless overridden)
security:
  - BearerAuth: []
```

| Security Type | `type` Value | Key Fields |
|--------------|-------------|------------|
| HTTP Bearer | `http` | `scheme: bearer`, `bearerFormat` |
| HTTP Basic | `http` | `scheme: basic` |
| API Key | `apiKey` | `in` (header/query/cookie), `name` |
| OAuth 2.0 | `oauth2` | `flows` (authorizationCode, etc.) |
| OpenID Connect | `openIdConnect` | `openIdConnectUrl` |
| Mutual TLS | `mutualTLS` | — (OpenAPI 3.1) |

**OAuth 2.0 Flows:**

| Flow | Use Case | Required Fields |
|------|----------|-----------------|
| `authorizationCode` | Web apps, SPAs, mobile | `authorizationUrl`, `tokenUrl`, `scopes` |
| `clientCredentials` | Machine-to-machine | `tokenUrl`, `scopes` |
| `implicit` (deprecated) | Legacy SPAs | `authorizationUrl`, `scopes` |
| `password` (deprecated) | Legacy first-party | `tokenUrl`, `scopes` |

**Rules:**
- Security schemes are defined in `components.securitySchemes` and referenced by name in `security` arrays. 
- The `security` field at the root level applies globally; at the operation level, it overrides the global setting. 
- An empty security array (`security: []`) makes security optional for that operation. 
- Multiple schemes in a single security requirement object are ANDed (all required); multiple objects in the array are ORed (any one suffices). 
- The `bearerFormat` field is a hint (e.g., `JWT`) and is not validated by tools. 
- API keys in query parameters should be used with caution; headers or cookies are preferred. 

### Annotated Code Example

```yaml
# security-schemes.yaml
openapi: 3.1.0
info:
  title: Secure API
  version: 1.0.0

# Global security: Bearer token required for all operations
security:
  - BearerAuth: []

paths:
  /public/health:
    get:
      summary: Health check (no auth required)
      security: []  # Override global — no security
      responses:
        '200':
          description: Service is healthy

  /admin/users:
    get:
      summary: List all users (admin only)
      security:
        - BearerAuth: []
          OAuth2: [admin]  # AND: both BearerAuth and OAuth2 admin scope required
      responses:
        '200':
          description: User list

  /api/data:
    get:
      summary: Access data (Bearer or API key)
      security:
        - BearerAuth: []
        - ApiKeyAuth: []  # OR: either BearerAuth or ApiKeyAuth
      responses:
        '200':
          description: Data

  /oauth/protected:
    get:
      summary: OAuth 2.0 protected endpoint
      security:
        - OAuth2: [read, write]  # Requires read AND write scopes
      responses:
        '200':
          description: Protected data

components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT access token

    ApiKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key

    OAuth2:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://auth.example.com/authorize
          tokenUrl: https://auth.example.com/token
          scopes:
            read: Read access
            write: Write access
            admin: Admin access
```

**Expected Output (rendered in Swagger UI):**
```
Authorize button: (JWT Bearer token)

GET /public/health — No auth
GET /admin/users — BearerAuth + OAuth2(admin)
GET /api/data — BearerAuth OR ApiKeyAuth
GET /oauth/protected — OAuth2(read, write)
```

**Why this output:** The global `security` applies BearerAuth to all operations. The `/public/health` operation overrides with `security: []` (no auth). The `/admin/users` operation requires both BearerAuth AND OAuth2 with the `admin` scope (AND logic). The `/api/data` operation accepts either BearerAuth OR ApiKeyAuth (OR logic). The `/oauth/protected` operation requires OAuth2 with both `read` and `write` scopes. Swagger UI renders the "Authorize" button for the global scheme and displays the security requirements per operation.

### Real-World Cases

- **Public APIs:** API keys in headers for service-to-service authentication.
- **User-facing APIs:** Bearer tokens (JWT) for authenticated user sessions.
- **Third-party integrations:** OAuth 2.0 authorization code flow with scopes.
- **Enterprise APIs:** OpenID Connect for SSO integration.
- **Zero-trust architectures:** Mutual TLS for service mesh authentication.

---

## References

- OpenAPI Specification v3.1.0 (Official) — https://spec.openapis.org/oas/v3.1.0.html
- OpenAPI Specification v3.0.3 (Official) — https://spec.openapis.org/oas/v3.0.3
- OpenAPI Initiative — https://www.openapis.org/
- Swagger 2.0 Specification — https://swagger.io/specification/v2/
- OpenAPI Version Comparison (ByJG) — https://opensource.byjg.com/pt/docs/php/swagger-test/version-comparison/
- OpenAPI vs Swagger: What's the Difference? (Mintlify) — https://www.mintlify.com/library/openapi-vs-swagger
- Swagger: OpenAPI vs Swagger (Postman Blog) — https://blog.postman.com/openapi-vs-swagger/
- OpenAPI Specification History (Bump.sh) — https://github.com/bump-sh/docs/blob/main/src/_specifications/openapi/v3.1/introduction/history.md
- Describing Parameters (Swagger.io) — https://swagger.io/docs/specification/v3_0/describing-parameters/
- Describing Request Body (Swagger.io) — https://swagger.io/docs/specification/v3_0/describing-request-body/describing-request-body/
- Reusing Descriptions (OpenAPI Learn) — https://learn.openapis.org/specification/components
- HTTP Methods (OpenAPI Learn) — https://learn.openapis.org/specification/http-methods
- Paths and Operations (Swagger.io) — https://swagger.io/docs/specification/v3_0/basic-structure/
- Responses Object (OpenAPI Specification) — https://spec.openapis.org/oas/v3.0.3#responses-object
- Security Scheme Object (OpenAPI Specification v3.1.0) — https://spec.openapis.org/oas/v3.1.0#security-scheme-object
- Describing API Security (OpenAPI Learn) — https://learn.openapis.org/specification/security
- Parameter Object (Redocly) — https://redocly.com/docs/openapi-visual-reference/parameter/
- Request Body Object (Redocly) — https://redocly.com/docs/openapi-visual-reference/request-body/
- References in OpenAPI documents (Microsoft Learn) — https://learn.microsoft.com/en-us/openapi/references
- OpenAPI 3.1 Webhooks (OpenAPI Learn) — https://learn.openapis.org/specification/webhooks
- x-webhooks Extension (Redocly) — https://redocly.com/docs/openapi-visual-reference/x-webhooks/
- RFC 7231 — HTTP/1.1 Semantics and Content — https://tools.ietf.org/html/rfc7231
- IANA HTTP Status Code Registry — https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
- JSON Schema Draft 2020-12 — https://json-schema.org/draft/2020-12/json-schema-core.html
- OpenAPI 3.2 Announcement — https://www.openapis.org/blog/2025/09/23/announcing-openapi-v3-2