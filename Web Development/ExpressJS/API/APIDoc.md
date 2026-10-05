# API Documentation Quality & SDKs — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** API documentation quality and SDK generation is the discipline of producing accurate, complete, and usable API reference documentation — including request/response examples, error catalogues, authentication guides, versioning strategies, and code snippets — and automatically generating client SDKs from that documentation.

**Technical Definition:** API documentation quality is measured by the completeness and accuracy of an OpenAPI Specification (OAS) document: every operation must have request body examples, response schemas, error responses (preferably standardised via RFC 7807 Problem Details), security scheme definitions, and versioning metadata. SDK generation uses tools like OpenAPI Generator to consume the OAS document and produce idiomatic, typed client libraries in multiple programming languages (JavaScript, Python, Java, Go, C#, etc.), eliminating manual boilerplate and ensuring clients remain in sync with the API contract.

**Beginner-Friendly Explanation:** Good API documentation is like a well-written instruction manual. It tells developers exactly how to use every endpoint, shows them example requests and responses in multiple formats, explains what happens when things go wrong, and provides ready-to-use code snippets they can copy and paste. SDK generation takes this one step further: instead of manually writing a JavaScript library, a Python library, and a Java library for your API, a tool reads your documentation and generates all of them automatically.

### Key Characteristics

- **Examples are mandatory:** Request and response examples (JSON, XML) reduce onboarding time and eliminate guesswork.
- **Error documentation is structured:** RFC 7807 (Problem Details) provides a standard, machine-readable error format.
- **Authentication must be documented:** Every security scheme (Bearer, API key, OAuth 2.0) needs clear, copy-pasteable instructions.
- **Versioning must be explicit:** URL, header, and query parameter versioning each have trade-offs; documentation must state which is used and how to select a version.
- **Code snippets accelerate adoption:** cURL, JavaScript, and Python examples cover the most common consumption patterns.
- **SDK generation is automated:** OpenAPI Generator produces client libraries for 50+ languages from a single specification.

### Prerequisites

- **A valid OpenAPI 3.0 or 3.1 specification** (JSON or YAML).
- **Basic understanding of HTTP:** Methods, status codes, headers, and the request–response cycle.
- **Familiarity with REST API design principles.**
- **For SDK generation:** OpenAPI Generator CLI installed (via npm or Docker) — Node.js v18 or later.

### Related Programming Areas

- **OpenAPI Specification:** The document format that drives documentation and SDK generation.
- **RFC 7807:** The standard for error response payloads.
- **Swagger UI / ReDoc:** Tools that render OpenAPI documents as interactive documentation.
- **OpenAPI Generator:** The primary tool for multi-language SDK generation.
- **API Governance:** Style guides and linting rules (Spectral, Redocly) that enforce documentation quality.

### Core Concepts

1. **Request and Response Body Examples** — JSON, XML, and multiple media types.
2. **Comprehensive Error Documentation** — Standardising error payloads like RFC 7807.
3. **Authentication and Authorization Guides** — Documenting security schemes.
4. **API Versioning Strategies** — URL, header, and query parameter versioning.
5. **Code Snippets for Consumers** — cURL, JavaScript, Python.
6. **SDK Generation** — Using OpenAPI Generator.

---

## Core Concept 1: Request and Response Body Examples (JSON, XML)

### Definitions

**Core Definition:** Request and response body examples are concrete illustrations of the data a client sends to or receives from an API endpoint, expressed in one or more media types (JSON, XML, form data, plain text).

**Technical Definition:** In OpenAPI 3.x, examples are attached to the `content` map under a media type key (e.g., `application/json`, `application/xml`). Each media type can include a `schema` (defining the structure) and an `examples` object (containing named examples with `summary` and `value` or `externalValue`). OpenAPI 3.0 and 3.1 support multiple media types per request or response, allowing the same endpoint to accept and return JSON and XML simultaneously.

**Beginner-Friendly Explanation:** Examples are the difference between telling someone "send a user object" and showing them exactly what that object looks like. OpenAPI lets you provide multiple examples — one for JSON, one for XML, one for a minimal request, one for a complete request — so developers can see exactly what the API expects and returns.

### Purposes

- To show developers exactly what data to send and what to expect back.
- To reduce onboarding time by providing copy-pasteable request/response payloads.
- To illustrate edge cases (minimal vs. complete requests, error responses).
- To support multiple media types (JSON, XML) with format-specific examples.
- To enable documentation tools to render interactive example tabs.

### Syntax Rules and Structure

```yaml
paths:
  /users:
    post:
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserInput'
            examples:
              minimal:
                summary: Minimal user
                value: { name: "Alice" }
              complete:
                summary: Complete user
                value: { name: "Bob", email: "bob@example.com", role: "admin" }
          application/xml:
            schema:
              $ref: '#/components/schemas/UserInput'
            examples:
              user:
                summary: User in XML
                externalValue: 'http://example.com/examples/user.xml'
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
              examples:
                standard:
                  summary: Standard response
                  value: { id: "uuid", name: "Alice", createdAt: "2026-01-15T10:00:00Z" }
```

| Component | Purpose |
|-----------|---------|
| `content` | Map of media types (e.g., `application/json`, `application/xml`). |
| `schema` | Defines the structure of the payload. |
| `examples` | Named examples with `summary` and `value` or `externalValue`. |
| `value` | Inline example (JSON/YAML object). |
| `externalValue` | URL to an external example file. |

**Rules:**
- Each media type can have its own schema and examples.
- `examples` is a map of named example objects; `example` (singular) is deprecated but still supported.
- `externalValue` allows referencing large examples stored externally.
- Multiple media types allow content negotiation (the client selects via `Accept` header).
- Response examples should cover success and error cases.

### Annotated Code Example

```yaml
# request-response-examples.yaml
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
                summary: Minimal request
                value:
                  name: "Alice"
              complete:
                summary: Complete request
                value:
                  name: "Bob"
                  email: "bob@example.com"
                  role: "admin"
          application/xml:
            schema:
              $ref: '#/components/schemas/UserInput'
            examples:
              user:
                summary: XML request
                value: |
                  <user>
                    <name>Alice</name>
                    <email>alice@example.com</email>
                  </user>
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
              examples:
                created:
                  summary: Created user
                  value:
                    id: "550e8400-e29b-41d4-a716-446655440000"
                    name: "Alice"
                    email: "alice@example.com"
                    createdAt: "2026-01-15T10:30:00Z"
        '422':
          description: Validation error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProblemDetails'
              examples:
                validation:
                  summary: Validation error
                  value:
                    type: "https://api.example.com/problems/validation-error"
                    title: "Validation Error"
                    status: 422
                    detail: "The request body failed validation"
                    errors:
                      - field: "email"
                        message: "Invalid email format"

components:
  schemas:
    UserInput:
      type: object
      required: [name]
      properties:
        name: { type: string }
        email: { type: string, format: email }
        role: { type: string, enum: [admin, user, viewer] }
    User:
      allOf:
        - $ref: '#/components/schemas/UserInput'
        - type: object
          properties:
            id: { type: string, format: uuid }
            createdAt: { type: string, format: date-time }
    ProblemDetails:
      type: object
      properties:
        type: { type: string, format: uri }
        title: { type: string }
        status: { type: integer }
        detail: { type: string }
```

**Expected Output (rendered in Swagger UI):**
```
POST /users
  Request Body (required):
    application/json:
      Examples:
        minimal: {"name": "Alice"}
        complete: {"name": "Bob", "email": "bob@example.com", "role": "admin"}
    application/xml:
      Example: <user><name>Alice</name>...</user>
  Responses:
    201: Created user — {"id": "...", "name": "Alice", ...}
    422: Validation error — ProblemDetails
```

**Why this output:** The `requestBody` defines two media types (JSON and XML), each with its own schema and examples. The JSON examples show a minimal request and a complete request. The 201 response includes a concrete example of the created user. The 422 response uses the `ProblemDetails` schema with a validation error example.

### Real-World Cases

- **Stripe:** Provides extensive JSON examples for every endpoint, including error responses.
- **Twilio:** Shows request and response examples in cURL, JSON, and XML.
- **Pinecone:** Uses multiple examples to illustrate vector operations.

---

## Core Concept 2: Comprehensive Error Documentation (RFC 7807)

### Definitions

**Core Definition:** RFC 7807 defines a "problem detail" as a way to carry machine-readable details of errors in an HTTP response, providing a standard format that avoids the need for each API to define its own error structure.

**Technical Definition:** RFC 7807 (Problem Details for HTTP APIs) defines a JSON object with five standard members: `type` (a URI identifying the problem type), `title` (a human-readable summary), `status` (the HTTP status code), `detail` (a human-readable explanation specific to this occurrence), and `instance` (a URI identifying the specific occurrence). The media type is `application/problem+json`. Extension members can add domain-specific fields.

**Beginner-Friendly Explanation:** RFC 7807 is a standard way to format error responses. Instead of every API returning errors in its own format — some with `{ "error": "..." }`, others with `{ "message": "..." }` — RFC 7807 defines a consistent structure with `type`, `title`, `status`, and `detail`. This means client code can handle errors from any RFC 7807-compliant API in the same way.

### Purposes

- To standardise error payloads across all endpoints and APIs.
- To provide machine-readable error details via the `type` URI.
- To include human-readable summaries (`title`) and specific explanations (`detail`).
- To enable client code to parse errors programmatically without endpoint-specific logic.
- To support extension members for domain-specific error context (e.g., validation errors).

### Syntax Rules and Structure

```yaml
components:
  schemas:
    ProblemDetails:
      type: object
      properties:
        type:
          type: string
          format: uri
          description: URI identifying the problem type
        title:
          type: string
          description: Human-readable summary
        status:
          type: integer
          description: HTTP status code
        detail:
          type: string
          description: Specific explanation for this occurrence
        instance:
          type: string
          format: uri
          description: URI identifying the specific occurrence
      required: [type, title, status]

responses:
  '422':
    description: Validation error
    content:
      application/problem+json:
        schema:
          $ref: '#/components/schemas/ProblemDetails'
        examples:
          validation:
            summary: Validation error
            value:
              type: "https://api.example.com/problems/validation-error"
              title: "Validation Error"
              status: 422
              detail: "The request body failed validation"
              instance: "/api/users/123"
              errors:
                - field: "email"
                  message: "Invalid email format"
```

| Member | Required | Description |
|--------|----------|-------------|
| `type` | Recommended | URI identifying the problem type. |
| `title` | Recommended | Human-readable summary. |
| `status` | Recommended | HTTP status code. |
| `detail` | Recommended | Specific explanation for this occurrence. |
| `instance` | Optional | URI identifying the specific occurrence. |
| Extensions | Optional | Domain-specific fields (e.g., `errors` array). |

**Rules:**
- The media type is `application/problem+json`.
- The `type` URI should be dereferenceable to human-readable documentation.
- The `status` member should mirror the HTTP status code.
- Extension members (e.g., `errors` for validation) provide domain-specific context.
- The `instance` member can identify the specific occurrence (e.g., request ID or resource URI).

### Annotated Code Example

```yaml
# rfc7807-error.yaml
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
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: User not found
          content:
            application/problem+json:
              schema:
                $ref: '#/components/schemas/ProblemDetails'
              examples:
                notFound:
                  summary: User not found
                  value:
                    type: "https://api.example.com/problems/user-not-found"
                    title: "User Not Found"
                    status: 404
                    detail: "No user exists with the specified ID"
                    instance: "/api/users/550e8400-e29b-41d4-a716-446655440000"

components:
  schemas:
    ProblemDetails:
      type: object
      properties:
        type: { type: string, format: uri }
        title: { type: string }
        status: { type: integer }
        detail: { type: string }
        instance: { type: string, format: uri }
      required: [type, title, status]
    User:
      type: object
      properties:
        id: { type: string, format: uuid }
        name: { type: string }
```

**Expected Output (rendered in Swagger UI):**
```
GET /users/{userId}
  Responses:
    200: User found — User
    404: User not found
      Content-Type: application/problem+json
      Example:
        {
          "type": "https://api.example.com/problems/user-not-found",
          "title": "User Not Found",
          "status": 404,
          "detail": "No user exists with the specified ID",
          "instance": "/api/users/550e8400-..."
        }
```

**Why this output:** The 404 response uses the `application/problem+json` media type and the `ProblemDetails` schema. The example shows the standard RFC 7807 members (`type`, `title`, `status`, `detail`, `instance`). Client code can parse this structure consistently across all error responses.

### Real-World Cases

- **Microsoft Graph:** Uses RFC 7807-style error responses with `error.code` and `error.message`.
- **Mews Loyalty Partner API:** Follows RFC 7807 as the standard for error responses.
- **Stripe:** Uses a similar but not identical structure (`error.type`, `error.code`, `error.message`).

---

## Core Concept 3: Authentication and Authorization Guides

### Definitions

**Core Definition:** Authentication and authorization documentation explains how clients prove their identity (authentication) and what resources or actions they are permitted to access (authorization), using OpenAPI security schemes.

**Technical Definition:** OpenAPI uses the term "security scheme" to cover both authentication and authorization schemes. OpenAPI 3.1 supports five security scheme types: HTTP authentication (Basic, Bearer), API keys (in headers, query, or cookies), OAuth 2.0 (authorizationCode, clientCredentials, implicit, password), OpenID Connect, and mutual TLS. Security schemes are defined in `components/securitySchemes` and applied globally or per-operation via the `security` field.

**Beginner-Friendly Explanation:** Security documentation tells developers how to authenticate with your API — whether to send a Bearer token in the `Authorization` header, an API key in `X-API-Key`, or go through an OAuth 2.0 flow to get a token. It also explains what each scheme allows (authorization), so developers know what their credentials can access.

### Purposes

- To document every authentication mechanism the API supports.
- To enable Swagger UI's "Authorize" button for interactive testing.
- To define OAuth 2.0 flows with authorization and token URLs.
- To specify scopes and permissions for authorization.
- To allow code generators to create authentication boilerplate.

### Syntax Rules and Structure

```yaml
components:
  securitySchemes:
    # API key in header
    ApiKeyHeader:
      type: apiKey
      in: header
      name: X-API-Key

    # API key in query (discouraged)
    ApiKeyQuery:
      type: apiKey
      in: query
      name: key

    # HTTP Basic authentication
    HttpBasicAuth:
      type: http
      scheme: basic

    # HTTP Bearer token (JWT)
    HttpBearerToken:
      type: http
      scheme: bearer
      bearerFormat: JWT

    # OAuth 2.0 with authorization code flow
    OAuth2ReadWrite:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://example.com/oauth/authorize
          tokenUrl: https://example.com/oauth/token
          refreshUrl: https://example.com/oauth/refresh
          scopes:
            read: Grants read access
            write: Grants write access

# Global security (applies to all operations)
security:
  - HttpBearerToken: []

paths:
  /bookings:
    get:
      summary: List bookings
      security:
        - OAuth2ReadWrite: [read]
      responses:
        '200':
          description: Booking list
```

| Security Type | `type` Value | Key Fields |
|--------------|-------------|------------|
| API Key | `apiKey` | `in` (header/query/cookie), `name` |
| HTTP Basic | `http` | `scheme: basic` |
| HTTP Bearer | `http` | `scheme: bearer`, `bearerFormat` |
| OAuth 2.0 | `oauth2` | `flows` with URLs and scopes |
| OpenID Connect | `openIdConnect` | `openIdConnectUrl` |
| Mutual TLS | `mutualTLS` | — |

**Rules:**
- API keys in query parameters are discouraged; headers are preferred.
- The `security` field at the root level applies globally; at the operation level, it overrides.
- An empty security array (`security: []`) makes security optional for that operation.
- Multiple schemes in one security requirement object are ANDed; multiple objects in the array are ORed.
- `bearerFormat` is a hint (e.g., `JWT`) and is not validated by tools.

### Annotated Code Example

```yaml
# security-schemes.yaml
openapi: 3.1.0
info:
  title: Secure API
  version: 1.0.0

security:
  - HttpBearerToken: []

paths:
  /public/health:
    get:
      summary: Health check (no auth)
      security: []
      responses:
        '200':
          description: Healthy

  /admin/users:
    get:
      summary: List users (admin)
      security:
        - OAuth2ReadWrite: [read, write]
      responses:
        '200':
          description: User list

  /api/data:
    get:
      summary: Data access (Bearer or API key)
      security:
        - HttpBearerToken: []
        - ApiKeyHeader: []
      responses:
        '200':
          description: Data

components:
  securitySchemes:
    HttpBearerToken:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT access token from /auth/login

    ApiKeyHeader:
      type: apiKey
      in: header
      name: X-API-Key
      description: API key for service-to-service calls

    OAuth2ReadWrite:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://auth.example.com/authorize
          tokenUrl: https://auth.example.com/token
          scopes:
            read: Read access
            write: Write access
```

**Expected Output (rendered in Swagger UI):**
```
Authorize button: JWT Bearer token
GET /public/health — No auth
GET /admin/users — OAuth2(read, write)
GET /api/data — Bearer OR API Key
```

**Why this output:** The global `security` applies Bearer token authentication to all operations. `/public/health` overrides with `security: []` (no auth). `/admin/users` requires OAuth2 with both `read` and `write` scopes. `/api/data` accepts either Bearer or API Key (OR logic). Swagger UI renders the "Authorize" button for the global scheme and displays security requirements per operation.

### Real-World Cases

- **Stripe:** Uses API keys in headers for authentication.
- **GitHub:** Uses OAuth 2.0 and personal access tokens (Bearer).
- **Google APIs:** Use OAuth 2.0 with scopes for authorization.

---

## Core Concept 4: API Versioning Strategies

### Definitions

**Core Definition:** API versioning is the practice of managing changes to an API's contract over time by exposing multiple versions simultaneously, allowing clients to migrate at their own pace.

**Technical Definition:** Three primary versioning strategies exist: **URL path versioning** (`/api/v1/resource`), **header versioning** (`X-API-Version: 1` or `Accept: application/json;api-version=1.0`), and **query parameter versioning** (`/api/resource?version=1`). Azure DevOps, for example, supports both header and query parameter versioning, with API versions formatted as `{major}.{minor}[-{stage}]` (e.g., `1.0`, `1.0-preview.1`).

**Beginner-Friendly Explanation:** API versioning is like having multiple editions of a book. The first edition (v1) still works for readers who have it, while the second edition (v2) is available for new readers. You can tell which edition you want by putting the version in the URL (`/v1/books`), in a header (`X-API-Version: 1`), or in a query parameter (`?version=1`).

### Purposes

- To introduce breaking changes without disrupting existing clients.
- To support multiple client generations during migration.
- To provide a clear deprecation timeline for old versions.
- To enable independent evolution of different API versions.

### Syntax Rules and Structure

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URL Path** | `/api/v1/users` | Simple, cache-friendly, visible | URL clutter |
| **Header** | `X-API-Version: 1` | Clean URLs | Hidden, requires client setup |
| **Query Parameter** | `/api/users?version=1` | Easy to test | Not RESTful, caching complexity |

**URL Path Versioning:**
```yaml
paths:
  /api/v1/users:
    get:
      summary: List users (v1)
      responses:
        '200':
          description: User list in v1 format
  /api/v2/users:
    get:
      summary: List users (v2)
      responses:
        '200':
          description: User list in v2 format
```

**Header Versioning:**
```yaml
paths:
  /api/users:
    get:
      summary: List users
      parameters:
        - name: X-API-Version
          in: header
          required: false
          schema:
            type: string
            enum: ['1', '2']
            default: '2'
      responses:
        '200':
          description: User list
```

**Query Parameter Versioning:**
```yaml
paths:
  /api/users:
    get:
      summary: List users
      parameters:
        - name: version
          in: query
          required: false
          schema:
            type: string
            enum: ['1', '2']
            default: '2'
      responses:
        '200':
          description: User list
```

**Rules:**
- URL path versioning is the most common and easiest to implement.
- Header versioning keeps URLs clean but requires client-side configuration.
- Query parameter versioning is easy to test in a browser but is not considered RESTful.
- Azure DevOps uses `{major}.{minor}[-{stage}]` format (e.g., `1.0-preview.1`).
- Document the versioning strategy in the API description.

### Annotated Code Example

```yaml
# versioning.yaml
openapi: 3.1.0
info:
  title: Versioned API
  version: 2.0.0
  description: |
    This API supports two versioning strategies:
    - URL path: /api/v1/... or /api/v2/...
    - Header: X-API-Version: 1 or 2

paths:
  /api/v1/users:
    get:
      summary: List users (v1)
      deprecated: true
      responses:
        '200':
          description: Flat array (v1 format)
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/UserV1'

  /api/v2/users:
    get:
      summary: List users (v2)
      responses:
        '200':
          description: Envelope with data and meta (v2 format)
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/UserV2'
                  meta:
                    $ref: '#/components/schemas/PaginationMeta'

components:
  schemas:
    UserV1:
      type: object
      properties:
        id: { type: integer }
        name: { type: string }
    UserV2:
      type: object
      properties:
        id: { type: string, format: uuid }
        name: { type: string }
        email: { type: string, format: email }
    PaginationMeta:
      type: object
      properties:
        page: { type: integer }
        limit: { type: integer }
        total: { type: integer }
```

**Expected Output (rendered in Swagger UI):**
```
GET /api/v1/users — List users (v1) — Deprecated
  Response: [{"id": 1, "name": "Alice"}]

GET /api/v2/users — List users (v2)
  Response: {"data": [{"id": "uuid", "name": "Alice", "email": "..."}], "meta": {...}}
```

**Why this output:** The v1 endpoint is marked `deprecated: true` and returns a flat array. The v2 endpoint returns an envelope with `data` and `meta`. The schemas differ (`UserV1` uses integer IDs; `UserV2` uses UUIDs and adds `email`). Clients can see the differences and plan their migration.

### Real-World Cases

- **Stripe:** URL-based versioning (`/v1/charges`) with date-based API versions.
- **GitHub:** Header-based versioning (`Accept: application/vnd.github.v3+json`).
- **Azure DevOps:** Both header and query parameter versioning (`api-version=1.0`).

---

## Core Concept 5: Code Snippets for Consumers (cURL, JavaScript, Python)

### Definitions

**Core Definition:** Code snippets are ready-to-use examples showing how to call an API endpoint in a specific programming language or tool, embedded directly in the documentation.

**Technical Definition:** The `x-codeSamples` OpenAPI extension provides custom code samples for an operation in one or more programming languages. Each sample includes a required `lang` string, a required `source` value containing the sample code, and an optional `label` for display. Documentation tools render each sample as a language-specific tab alongside the operation. Tools like `openapi-snippet` generate snippets automatically from the specification for cURL, Node.js, Python, Ruby, Java, Go, C#, and more.

**Beginner-Friendly Explanation:** Code snippets are copy-pasteable examples that show developers exactly how to call your API in their preferred language. Instead of reading the documentation and figuring out how to make the HTTP request themselves, they can copy a cURL command or a JavaScript `fetch` call and run it immediately.

### Purposes

- To reduce time-to-first-call by providing ready-to-use examples.
- To show idiomatic usage in multiple languages (cURL, JavaScript, Python).
- To demonstrate authentication, headers, and request bodies in context.
- To enable developers to test endpoints without writing code from scratch.

### Syntax Rules and Structure

```yaml
paths:
  /pets:
    get:
      summary: List pets
      x-codeSamples:
        - lang: cURL
          label: CLI
          source: |
            curl --request GET \
              --url https://api.example.com/pets \
              --header 'accept: application/json' \
              --header 'Authorization: Bearer <token>'
        - lang: JavaScript
          label: fetch
          source: |
            const response = await fetch("https://api.example.com/pets", {
              headers: {
                accept: "application/json",
                Authorization: "Bearer <token>"
              }
            });
            console.log(await response.json());
        - lang: Python
          label: requests
          source: |
            import requests
            response = requests.get(
                "https://api.example.com/pets",
                headers={"Authorization": "Bearer <token>"}
            )
            print(response.json())
```

| Field | Required | Description |
|-------|----------|-------------|
| `lang` | Yes | Programming language or syntax highlighter. |
| `source` | Yes | The sample code (inline string or `$ref`). |
| `label` | No | Display label for the sample. |

**Rules:**
- `x-codeSamples` is the successor to `x-code-samples` (camelCase).
- `source` can be an inline string or a `$ref` to an external file.
- Documentation tools render each sample as a tab or code block.
- Tools like `openapi-snippet` can generate snippets automatically from the spec.
- Include authentication headers in the examples.

### Annotated Code Example

```yaml
# code-samples.yaml
openapi: 3.1.0
info:
  title: Pet API
  version: 1.0.0

paths:
  /pets:
    get:
      summary: List pets
      operationId: listPets
      x-codeSamples:
        - lang: cURL
          label: cURL
          source: |
            curl --request GET \
              --url https://api.example.com/pets \
              --header 'accept: application/json' \
              --header 'Authorization: Bearer <token>'
        - lang: JavaScript
          label: JavaScript (fetch)
          source: |
            const response = await fetch("https://api.example.com/pets", {
              method: "GET",
              headers: {
                accept: "application/json",
                Authorization: "Bearer <token>"
              }
            });
            const pets = await response.json();
            console.log(pets);
        - lang: Python
          label: Python (requests)
          source: |
            import requests

            headers = {
                "accept": "application/json",
                "Authorization": "Bearer <token>"
            }
            response = requests.get(
                "https://api.example.com/pets",
                headers=headers
            )
            print(response.json())
      responses:
        '200':
          description: Pet list
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
      properties:
        id: { type: integer }
        name: { type: string }
```

**Expected Output (rendered in Swagger UI):**
```
GET /pets — List pets
  Code Samples:
    [cURL] [JavaScript (fetch)] [Python (requests)]

    cURL:
      curl --request GET \
        --url https://api.example.com/pets \
        --header 'accept: application/json' \
        --header 'Authorization: Bearer <token>'

    JavaScript (fetch):
      const response = await fetch("https://api.example.com/pets", { ... });

    Python (requests):
      import requests
      response = requests.get("https://api.example.com/pets", ...)
```

**Why this output:** The `x-codeSamples` extension provides three code samples (cURL, JavaScript, Python) for the `GET /pets` operation. Swagger UI renders them as tabs, allowing developers to switch between languages. Each sample includes the full request including headers and authentication.

### Real-World Cases

- **Stripe:** Provides code samples in cURL, Ruby, Python, PHP, Java, Node.js, and Go.
- **Twilio:** Shows cURL, Python, Node.js, PHP, Ruby, Java, and C# examples.
- **GitHub:** Provides cURL examples for every endpoint.

---

## Core Concept 6: SDK Generation (Using OpenAPI Generator)

### Definitions

**Core Definition:** SDK generation is the automated process of producing client libraries (SDKs) in multiple programming languages from an OpenAPI specification, eliminating manual boilerplate and ensuring consistency.

**Technical Definition:** OpenAPI Generator is a tool that generates API client libraries (SDK generation), server stubs, documentation, and configuration automatically given an OpenAPI Spec (v2, v3). It supports 50+ languages and frameworks, including ActionScript, Ada, Apex, Bash, C, C#, C++, Clojure, Dart, Elixir, Erlang, Go, Groovy, Haskell, Java, JavaScript, Kotlin, Lua, Node.js, Objective-C, Perl, PHP, PowerShell, Python, R, Ruby, Rust, Scala, Swift, TypeScript, and more.

**Beginner-Friendly Explanation:** Instead of manually writing a JavaScript library, a Python library, and a Java library for your API — and keeping them all in sync — you run a single command that reads your OpenAPI specification and generates all of them automatically. The generated SDKs include typed methods for every endpoint, authentication handling, and error classes.

### Purposes

- To generate client SDKs in multiple languages from a single specification.
- To eliminate manual boilerplate and reduce integration effort for consumers.
- To ensure SDKs remain in sync with the API contract.
- To generate server stubs for rapid prototyping and migration.
- To provide consistent, idiomatic client libraries across languages.

### Syntax Rules and Structure

**Installation:**
```bash
# Via npm
npm install @openapitools/openapi-generator-cli -g

# Via Docker
docker pull openapitools/openapi-generator-cli
```

**Generate a TypeScript SDK:**
```bash
openapi-generator-cli generate \
  -i ./openapi.yaml \
  -g typescript-fetch \
  -o ./generated/typescript-sdk \
  --additional-properties=npmName=my-api-client,supportsES6=true
```

**Generate a Python SDK:**
```bash
openapi-generator-cli generate \
  -i ./openapi.yaml \
  -g python \
  -o ./generated/python-sdk \
  --additional-properties=packageName=my_api_client
```

**Generate a Java SDK:**
```bash
openapi-generator-cli generate \
  -i ./openapi.yaml \
  -g java \
  -o ./generated/java-sdk \
  --library=okhttp-gson \
  --additional-properties=hideGenerationTimestamp=true
```

| Command Component | Description |
|-------------------|-------------|
| `-i` | Input OpenAPI specification file. |
| `-g` | Generator name (e.g., `typescript-fetch`, `python`, `java`). |
| `-o` | Output directory. |
| `--library` | Specific HTTP library (e.g., `okhttp-gson` for Java). |
| `--additional-properties` | Generator-specific configuration. |

**Rules:**
- The specification must be valid OpenAPI 2.0, 3.0, or 3.1.
- Each generator has its own configuration options (`--additional-properties`).
- Generated SDKs include model classes, API classes, authentication handlers, and error types.
- SDKs should be regenerated whenever the specification changes.
- The `--library` option selects the HTTP client implementation for the target language.

### Annotated Code Example

```bash
# sdk-generation.sh

# Generate TypeScript SDK with fetch
openapi-generator-cli generate \
  -i ./openapi.yaml \
  -g typescript-fetch \
  -o ./sdk/typescript \
  --additional-properties=npmName=@mycompany/api-client,supportsES6=true

# Generate Python SDK
openapi-generator-cli generate \
  -i ./openapi.yaml \
  -g python \
  -o ./sdk/python \
  --additional-properties=packageName=my_api_client

# Generate Java SDK with okhttp-gson
openapi-generator-cli generate \
  -i ./openapi.yaml \
  -g java \
  -o ./sdk/java \
  --library=okhttp-gson \
  --additional-properties=hideGenerationTimestamp=true

# Generate Go SDK
openapi-generator-cli generate \
  -i ./openapi.yaml \
  -g go \
  -o ./sdk/go \
  --additional-properties=packageName=myapiclient
```

**Expected Output (generated TypeScript SDK structure):**
```
sdk/typescript/
├── src/
│   ├── apis/
│   │   ├── UsersApi.ts
│   │   └── PetsApi.ts
│   ├── models/
│   │   ├── User.ts
│   │   └── Pet.ts
│   ├── runtime.ts
│   └── index.ts
├── package.json
└── tsconfig.json
```

**Expected Output (generated Python SDK structure):**
```
sdk/python/
├── my_api_client/
│   ├── api/
│   │   ├── users_api.py
│   │   └── pets_api.py
│   ├── models/
│   │   ├── user.py
│   │   └── pet.py
│   ├── api_client.py
│   └── configuration.py
├── setup.py
└── README.md
```

**Why this output:** The generator reads the OpenAPI specification and produces a complete SDK in each language. Each SDK includes API classes with methods for every operation, model classes for every schema, authentication configuration, and error handling. The generated code is idiomatic to each language and can be published to package registries (npm, PyPI, Maven Central).

### Real-World Cases

- **VMware:** Generates custom .NET C# API clients from OpenAPI specs using OpenAPI Generator.
- **Stripe:** Provides official SDKs in multiple languages, many generated from their OpenAPI spec.
- **Kubernetes:** Generates client libraries from its OpenAPI specification.

---

## References

- RFC 7807 — Problem Details for HTTP APIs — https://datatracker.ietf.org/doc/html/rfc7807
- OpenAPI Specification v3.1.0 — https://spec.openapis.org/oas/v3.1.0.html
- x-codeSamples Extension Registry — https://spec.openapis.org/registry/extension/x-codeSamples.html
- OpenAPI Generator FAQ: Generators — https://openapi-generator.tech/docs/faq-generators/
- OpenAPI Generator CLI Installation — https://openapi-generator.tech/docs/installation/
- OpenAPI Generator GitHub — https://github.com/OpenAPITools/openapi-generator
- Describing API Security (OpenAPI Guide) — https://docs.bump.sh/openapi/v3.2/advanced/security/
- REST API Versioning for Azure DevOps — https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/rest-api-versioning
- Mintlify: API Documentation Recommendations — https://www.mintlify.com/library/our-recommendations-for-creating-api-documentation-with-examples
- openapi-snippet GitHub — https://github.com/confluentinc/openapi-snippet
- Redocly: codeSamples Configuration — https://redocly.com/docs/api-reference-docs/configuration/functionality/
- OpenAPI Generator Gradle Plugin — https://plugins.gradle.org/plugin/org.openapi.generator
- IBM: OpenAPI Request Body Examples — https://www.ibm.com/docs/en/app-connect/12.0.x?topic=reference-openapi-request-body
- Swagger: Adding Examples — https://swagger.io/docs/specification/v3_0/adding-examples/