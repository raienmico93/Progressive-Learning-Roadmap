# Environments & Postman/Insomnia Integration — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Environment management in API tooling is the practice of isolating configuration values — base URLs, API keys, tokens, and other environment-specific settings — into named, switchable variable groups so that the same request collection or API specification can target development, staging, and production servers without modification.

**Technical Definition:** Postman environments are named sets of key-value variable pairs that can be referenced in requests, pre-request scripts, and test scripts using `{{variableName}}` syntax. When you switch the active environment, all variables in requests and scripts resolve to the values from that environment. Insomnia provides an equivalent mechanism through "Base Environment" and sub-environments, where variables are referenced with the same `{{variableName}}` mustache syntax. OpenAPI specifications can be imported into both tools, automatically generating request collections and environment templates from the `servers` array in the specification.

**Beginner-Friendly Explanation:** Imagine you have a set of API requests that you use for testing. You want to run them against your local development server, your staging server, and your production server. Instead of manually changing the URL in every request each time, you create three "environments" — one for each server — each containing the base URL and any API keys. You switch environments with a single dropdown, and every request automatically uses the right server.

### Key Characteristics

- **Variable scoping:** Postman supports global, collection, environment, and local variables; Insomnia supports base environments and sub-environments.
- **Secret masking:** Postman allows variables to be marked as "secret" (masked in UI and logs); Insomnia has similar capabilities.
- **OpenAPI-driven generation:** Importing an OpenAPI spec automatically creates collections and environment templates in both tools.
- **Environment inheritance:** Insomnia sub-environments inherit from the base environment; Postman environments can be shared across a team.
- **Mock server integration:** Prism can consume the same OpenAPI spec that drives Postman/Insomnia collections, providing a local mock server.

### Prerequisites

- **Postman** (desktop or web) or **Insomnia** (desktop) installed.
- **A valid OpenAPI 3.0/3.1 specification** (JSON or YAML).
- **Node.js** (v18 or higher) for Prism CLI.
- **Basic understanding of API request structure:** URLs, headers, authentication.

### Related Programming Areas

- **OpenAPI Specification:** The source document for collection and environment generation.
- **API Mocking:** Prism and similar tools consume the same OpenAPI spec.
- **CI/CD Pipelines:** Environments enable automated testing across stages.
- **API Client Tools:** Postman and Insomnia are the dominant desktop API clients.
- **Secret Management:** Environment variables isolate credentials from version control.

### Core Concepts

1. **Environment Management** — Development, Staging, Production URLs.
2. **Exporting OpenAPI Specs to Postman Collections** — Conversion tools and workflows.
3. **Importing OpenAPI Specs into Insomnia** — UI and CLI import methods.
4. **Mocking Express APIs** — Using Prism to generate mock servers from documentation.

---

## Core Concept 1: Environment Management

### Definitions

**Core Definition:** Environment management is the practice of organising API configuration values into named, switchable variable groups so that the same request collection can target multiple deployment environments.

**Technical Definition:** In Postman, an environment is a set of one or more variables that can be referenced when sending requests, writing pre-request scripts, or writing test scripts. You can create environments for different types of work — development, staging, production — and when you switch between environments, all variables in your requests and scripts use the values from the current environment. Insomnia uses a similar model: a "Base Environment" provides shared variables, and "Sub Environments" override specific values for each context.

**Beginner-Friendly Explanation:** Think of environments as different "profiles" for your API tool. Each profile contains a base URL and authentication credentials. You switch profiles with a dropdown, and all your requests automatically use the right server.

### Purposes

- To avoid hardcoding URLs, API keys, and tokens in every request.
- To test the same collection against development, staging, and production servers with a single click.
- To isolate sensitive credentials from version control by keeping them in environment files.
- To enable team collaboration by sharing environment definitions (with secrets masked).
- To support CI/CD pipelines by injecting environment values at runtime.

### Sub-Feature 1.1: Postman Environment Management

#### Syntax Rules and Structure

**Creating an Environment:**
```
Environments (sidebar) → + → Name: "Development"
→ Add variables:
   base_url = http://localhost:3000
   auth_token = dev-token-xyz (mark as Secure)
```

| Variable Type | Scope | Use Case |
|--------------|-------|----------|
| Global | All requests in workspace | Shared config across collections |
| Collection | All requests in a collection | Collection-specific values |
| Environment | Active environment only | Dev/staging/prod URLs and credentials |
| Local | Single request execution | Temporary values |

**Referencing Variables:**
```http
GET {{base_url}}/api/users
Authorization: Bearer {{auth_token}}
```

**Environment Inheritance (Postman):**
```json
// Staging environment
{
  "values": [
    { "key": "base_url", "value": "https://staging-api.example.com", "enabled": true },
    { "key": "auth_token", "value": "{{getStagingToken}}", "type": "secret" }
  ],
  "_inherits": { "id": "base-env-id" }
}
```

**Rules:**
- Environment variables are referenced with `{{variableName}}` (double curly braces).
- Mark sensitive values as **Secure** to mask them in the UI and logs.
- Use the environment selector in the top-right corner to switch environments.
- Changes to variables are automatically saved.
- Postman Vault can store secrets locally or share them with your team while keeping sensitive values masked.

#### Annotated Code Example

```json
// dev-environment.json
{
  "name": "Development",
  "values": [
    { "key": "base_url", "value": "http://localhost:3000", "enabled": true },
    { "key": "auth_token", "value": "dev-token-xyz", "enabled": true, "type": "secret" },
    { "key": "callback_url", "value": "http://localhost:3000/callback", "enabled": true }
  ]
}
```

```json
// staging-environment.json
{
  "name": "Staging",
  "values": [
    { "key": "base_url", "value": "https://staging-api.example.com", "enabled": true },
    { "key": "auth_token", "value": "staging-token-abc", "enabled": true, "type": "secret" },
    { "key": "callback_url", "value": "https://staging-api.example.com/callback", "enabled": true }
  ]
}
```

**Expected Output (request execution):**
```
GET http://localhost:3000/api/users    (Development environment active)
GET https://staging-api.example.com/api/users   (Staging environment active)
```

**Why this output:** The `{{base_url}}` variable resolves to the value in the currently active environment. When you switch from Development to Staging, every request using `{{base_url}}` automatically targets the staging server. The `auth_token` variable is marked as `secret`, so its value is masked in the Postman UI and logs.

---

### Sub-Feature 1.2: Insomnia Environment Management

#### Syntax Rules and Structure

**Base Environment (shared variables):**
```json
{
  "base_url": "{{ scheme }}://{{ host }}{{ base_path }}",
  "scheme": "https",
  "host": "api.example.com",
  "base_path": "/v1"
}
```

**Sub Environment (overrides):**
```json
{
  "host": "staging-api.example.com"
}
```

**Referencing Variables:**
```http
GET {{ base_url }}/users
```

**After-Response Script (set variable from response):**
```js
insomnia.test('Check if status is 201', () => {
  insomnia.expect(insomnia.response.code).to.eql(201);
  if (insomnia.response.code) {
    const jsonBody = insomnia.response.json();
    insomnia.environment.set("systemToken", jsonBody.token);
  }
});
```

**Rules:**
- Insomnia uses `{{ variable }}` syntax (spaces optional).
- The Base Environment is shared across the workspace; sub-environments override specific values.
- Insomnia automatically creates an "OpenAPI env" sub-environment when you import an OpenAPI specification, with variables derived from the `servers` array.
- After-response scripts can set environment variables from response values (e.g., tokens).
- The environment selector is at the top-left of the Insomnia interface.

#### Annotated Code Example

```json
// Insomnia base environment
{
  "base_url": "{{ scheme }}://{{ host }}{{ base_path }}",
  "scheme": "https",
  "host": "api.example.com",
  "base_path": "/v1",
  "bearerToken": "your-personal-access-token"
}
```

```json
// Insomnia sub-environment: "Staging"
{
  "host": "staging-api.example.com"
}
```

```js
// After-response script to capture a token
insomnia.test('Check if status is 201', () => {
  insomnia.expect(insomnia.response.code).to.eql(201);
  if (insomnia.response.code) {
    const jsonBody = insomnia.response.json();
    insomnia.environment.set("systemToken", jsonBody.token);
  }
});
```

**Expected Output:**
```
GET https://api.example.com/v1/users          (Base environment)
GET https://staging-api.example.com/v1/users  (Staging sub-environment)
```

**Why this output:** The base environment defines `base_url` as a composed variable (`scheme://host/base_path`). The Staging sub-environment overrides only `host`, so `base_url` resolves to the staging host while keeping the scheme and base path. The after-response script captures a token from a response and stores it as `systemToken` for use in subsequent requests.

---

## Core Concept 2: Exporting OpenAPI Specs to Postman Collections

### Definitions

**Core Definition:** Exporting an OpenAPI specification to a Postman Collection means converting the specification into Postman's Collection v2.1 format, which Postman can import to generate requests, folders, and environment templates automatically.

**Technical Definition:** The official `openapi-to-postman` converter (npm package `openapi-to-postmanv2`) converts OpenAPI 3.0, 3.1, and Swagger 2.0 specifications into Postman Collection v2.1 format. It supports both CLI usage and Node.js module usage. The converter parses the OpenAPI document's paths, operations, parameters, request bodies, and responses to generate corresponding Postman requests with headers, query parameters, and body payloads pre-configured.

**Beginner-Friendly Explanation:** You have an OpenAPI specification that describes your API. Postman can't directly use that file — it needs its own collection format. The `openapi-to-postman` converter reads your spec and produces a Postman collection file that you can import. All your endpoints appear as requests, grouped by tags, with parameters and bodies already filled in.

### Purposes

- To generate a Postman collection from an existing OpenAPI specification without manual work.
- To keep Postman collections in sync with the API contract as the spec evolves.
- To enable teams to test APIs using Postman with requests pre-configured from the spec.
- To support CI/CD pipelines where collections are generated and run automatically.

### Syntax Rules and Structure

**Installation:**
```bash
npm install -g openapi-to-postmanv2
```

**CLI Usage:**
```bash
openapi2postmanv2 \
  -s ./openapi.yaml \
  -o ./postman-collection.json \
  -p \
  --options ./options.json
```

| CLI Option | Description |
|-----------|-------------|
| `-s <source>` | Path to the OpenAPI specification file. |
| `-o <destination>` | Path for the output Postman collection. |
| `-p` | Pretty-print the collection JSON. |
| `-O <options>` | Path to a JSON options file for conversion settings. |

**Node.js Module Usage:**
```js
const converter = require('openapi-to-postmanv2');

converter.convert(
  { type: 'file', data: './openapi.yaml' },
  {
    requestNameSource: 'Fallback',
    indentCharacter: 'Space',
    folderStrategy: 'Tags'
  },
  (err, result) => {
    if (err) return console.error(err);
    if (result.result) {
      fs.writeFileSync('postman-collection.json', JSON.stringify(result.output[0].data));
    }
  }
);
```

**Common Options:**

| Option | Values | Description |
|--------|--------|-------------|
| `requestNameSource` | `'Fallback'`, `'URL'` | How request names are generated. |
| `indentCharacter` | `'Space'`, `'Tab'` | Indentation in output JSON. |
| `folderStrategy` | `'Paths'`, `'Tags'` | How requests are grouped into folders. |
| `optimizeConversion` | `true`/`false` | Improve conversion quality. |

**Rules:**
- The converter supports OpenAPI 3.0, 3.1, and Swagger 2.0.
- The output is a Postman Collection v2.1 JSON file.
- Import the generated collection into Postman via **File → Import → Upload File**.
- Use the `-p` flag to pretty-print the output for easier version control.
- The `folderStrategy: 'Tags'` option groups requests by OpenAPI tags, matching the organisation in Swagger UI.

### Annotated Code Example

```bash
# Step 1: Install the converter globally
npm install -g openapi-to-postmanv2

# Step 2: Convert the OpenAPI spec to a Postman collection
openapi2postmanv2 \
  -s ./openapi.yaml \
  -o ./postman-collection.json \
  -p \
  -O ./postman-options.json

# Step 3: Import into Postman
# Postman → File → Import → Upload File → postman-collection.json
```

```json
// postman-options.json
{
  "requestNameSource": "Fallback",
  "indentCharacter": "Space",
  "folderStrategy": "Tags",
  "optimizeConversion": true
}
```

**Expected Output (Postman after import):**
```
Collection: "My API"
├── Users (folder, from tag "Users")
│   ├── GET /users — List users
│   ├── GET /users/{id} — Get user by ID
│   └── POST /users — Create user
└── Products (folder, from tag "Products")
    ├── GET /products — List products
    └── POST /products — Create product
```

**Why this output:** The converter reads the OpenAPI spec and generates a Postman collection with folders organised by tags (`folderStrategy: 'Tags'`). Each operation becomes a request with the correct HTTP method, URL path, headers, query parameters, and request body pre-configured from the spec. Importing the collection into Postman makes all endpoints immediately testable.

---

## Core Concept 3: Importing OpenAPI Specs into Insomnia

### Definitions

**Core Definition:** Importing an OpenAPI specification into Insomnia means loading the specification into Insomnia as a "Design Document" or collection, which automatically generates requests, folders, and an environment template from the spec's paths and servers.

**Technical Definition:** Insomnia supports importing OpenAPI 3.0 and 3.1 specifications (JSON or YAML) from a file, URL, or clipboard. During import, Insomnia creates a Design Document with all endpoints as requests, grouped by tags if present. It also creates an "OpenAPI env" sub-environment with variables derived from the `servers` array in the specification (e.g., `scheme`, `host`, `base_path`). The Inso CLI provides command-line import, validation, and test execution for CI/CD integration.

**Beginner-Friendly Explanation:** Insomnia can read your OpenAPI file and turn it into a ready-to-use collection of API requests. It also creates an environment with the server URL from your spec, so you can start testing immediately. If you prefer the command line, the Inso CLI can do the same thing and integrate with your CI pipeline.

### Purposes

- To generate Insomnia requests from an OpenAPI specification without manual setup.
- To automatically create environment variables from the spec's `servers` array.
- To validate OpenAPI specifications in CI using `inso lint spec`.
- To run Insomnia test suites from the command line with `inso run test`.
- To export the raw OpenAPI spec from a Design Document using `inso export spec`.

### Syntax Rules and Structure

**UI Import:**
```
Insomnia → Create → Import from File/URL/Clipboard
→ Select OpenAPI 3.0/3.1 (JSON or YAML)
→ Insomnia generates:
   - Requests for every operation
   - Folders grouped by tags
   - "OpenAPI env" sub-environment with server variables
```

**CLI Import (Inso CLI):**
```bash
# Import from URL
inso import --from-url https://api.example.com/openapi.json --output ./insomnia-import.json

# Export OpenAPI spec from a Design Document
inso export spec "My API" --output ./exported-spec.yaml

# Lint the OpenAPI spec (fail CI on errors)
inso lint spec "My API"

# Run test suites from CLI
inso run test "My API" --env "Staging"
```

| Inso Command | Purpose |
|-------------|---------|
| `inso import` | Import OpenAPI/Postman/HAR into Insomnia format. |
| `inso export spec` | Export the OpenAPI spec from a Design Document. |
| `inso lint spec` | Validate the OpenAPI spec (exit code on errors). |
| `inso run test` | Execute test suites defined in Insomnia. |

**Environment Variables from OpenAPI:**
```
Insomnia creates an "OpenAPI env" sub-environment with:
{
  "scheme": "https",
  "host": "api.example.com",
  "base_path": "/v1",
  "base_url": "{{ scheme }}://{{ host }}{{ base_path }}"
}
```

**Rules:**
- Insomnia supports OpenAPI 3.0/3.1, Swagger, Postman collections, HAR, WSDL, and cURL.
- The "OpenAPI env" sub-environment is created automatically during import.
- The `base_url` variable is composed from `scheme`, `host`, and `base_path`.
- Use `inso lint spec` in CI to validate the OpenAPI spec and fail builds on errors.
- The Inso CLI `run test` command returns a non-zero exit code if tests fail, making it CI-friendly.
- During imports and exports, Insomnia replaces real UUIDs with special resource IDs (e.g., `__WORKSPACE_ID__`) to preserve structure and prevent collisions.

### Annotated Code Example

```bash
# Step 1: Import OpenAPI spec into Insomnia via CLI
inso import \
  --from-url https://api.example.com/openapi.json \
  --output ./insomnia-workspace.json

# Step 2: Validate the spec in CI
inso lint spec "My API" || exit 1

# Step 3: Run tests against staging environment
inso run test "My API" --env "Staging"

# Step 4: Export the spec for version control
inso export spec "My API" --output ./openapi-exported.yaml
```

**Expected Output (Insomnia after import):**
```
Design Document: "My API"
├── Users (folder)
│   ├── GET /users — List users
│   ├── GET /users/{id} — Get user by ID
│   └── POST /users — Create user
├── Products (folder)
│   └── ...
Environments:
  Base Environment
    └── OpenAPI env (scheme, host, base_path, base_url)
```

**Why this output:** The import creates a Design Document with requests organised by tags, and an "OpenAPI env" sub-environment with variables derived from the `servers` array in the specification. The `base_url` variable is composed as `{{ scheme }}://{{ host }}{{ base_path }}`, allowing you to switch environments by overriding just the `host` variable.

---

## Core Concept 4: Mocking Express APIs Using Documentation (Prism)

### Definitions

**Core Definition:** Prism is an open-source HTTP mock server that generates a fully functional mock API from an OpenAPI v2/v3 (or Postman Collection) document, allowing frontend and mobile teams to develop against a simulated API before the real backend exists.

**Technical Definition:** Prism is a set of packages for API mocking with OpenAPI v2 (Swagger) and OpenAPI v3. The CLI (`@stoplight/prism-cli`) reads an OpenAPI document and starts an HTTP server that responds to requests with mock data generated from the schemas, examples, and definitions in the document. Prism validates incoming requests against the specification and returns realistic responses. It can be run locally or in CI, and supports dynamic response generation based on schema definitions.

**Beginner-Friendly Explanation:** You have an OpenAPI specification that describes your API — every endpoint, every parameter, every response schema. Prism reads that file and instantly creates a working mock server. Frontend developers can call the mock server and get realistic responses, even before the backend is built. The mock server validates requests against the spec and returns the correct data structure.

### Purposes

- To provide a working API for frontend/mobile teams before the backend is implemented.
- To test against a mock that validates requests and returns spec-compliant responses.
- To avoid costs and rate limits associated with calling real third-party APIs (e.g., Twilio).
- To enable offline development without internet connectivity.
- To simulate endpoints that are still under development or in private beta.

### Syntax Rules and Structure

**Installation:**
```bash
npm install -g @stoplight/prism-cli
# OR
yarn global add @stoplight/prism-cli
```

**Starting a Mock Server:**
```bash
prism mock openapi.yaml
# → Prism is listening on http://127.0.0.1:4010
```

**Starting with a Remote Spec:**
```bash
prism mock https://raw.githubusercontent.com/twilio/twilio-oai/main/spec/json/twilio_api_v2010.json
```

| Command | Purpose |
|---------|---------|
| `prism mock <spec>` | Start a mock server from an OpenAPI spec. |
| `prism proxy <spec> <url>` | Start a validating proxy in front of a real API. |
| `prism validate <spec>` | Validate an OpenAPI spec without starting a server. |

**Rules:**
- Prism requires Node.js >= 18.16 for Node.js 18.x, and Node.js >= 16 for older versions.
- Prism supports OpenAPI v2 (Swagger) and OpenAPI v3.
- The mock server validates incoming requests against the spec and returns errors for invalid requests.
- Prism uses examples from the spec if provided; otherwise, it generates values from schemas (e.g., `string` for strings, `0` for numbers).
- The mock server is stateless — repeated requests return the same response.
- Prism can also validate requests/responses as a proxy in front of a real API.

### Annotated Code Example

```bash
# Step 1: Install Prism CLI
npm install -g @stoplight/prism-cli

# Step 2: Start a mock server from your OpenAPI spec
prism mock ./openapi.yaml

# Output:
# [CLI] ...  awaiting  Starting Prism...
# [CLI] ℹ  info      Prism is listening on http://127.0.0.1:4010
# [CLI] ℹ  info      GET        http://127.0.0.1:4010/users
# [CLI] ℹ  info      POST       http://127.0.0.1:4010/users
# [CLI] ℹ  info      GET        http://127.0.0.1:4010/users/{id}
```

```bash
# Step 3: Test the mock server
curl http://127.0.0.1:4010/users/1
# Response (generated from schema):
# {"id": 0, "name": "string", "email": "string"}
```

**Expected Output (Prism console):**
```
[CLI] ...  awaiting  Starting Prism...
[CLI] ℹ  info      Prism is listening on http://127.0.0.1:4010
[CLI] ℹ  info      GET        http://127.0.0.1:4010/users
[CLI] ℹ  info      POST       http://127.0.0.1:4010/users
[CLI] ℹ  info      GET        http://127.0.0.1:4010/users/{id}
```

**Expected Output (curl response):**
```json
{"id": 0, "name": "string", "email": "string"}
```

**Why this output:** Prism reads the OpenAPI spec and starts an HTTP server on port 4010. Every endpoint defined in the spec is available. When a request arrives, Prism validates it against the spec (checking parameters, request body, etc.) and returns a response generated from the schema. If the spec includes `examples`, Prism returns the example values; otherwise, it generates placeholder values (`0` for numbers, `"string"` for strings).

### Real-World Cases

- **Frontend development:** Mobile and web teams develop against the mock while the backend is being built.
- **Third-party API simulation:** Twilio uses Prism to let developers test against a mock of the Twilio API without incurring costs.
- **CI/CD testing:** Mock servers run in CI pipelines for fast, deterministic integration tests.
- **Offline development:** Developers can work without internet access by running a local mock.

---

## References

- Postman Docs: Group sets of variables in Postman using environments — https://learning.postman.com/docs/use/send-requests/variables/managing-environments
- Postman Docs: Store and reuse values using variables — https://learning.postman.com/docs/use/send-requests/variables/variables/
- openapi-to-postman GitHub Repository — https://github.com/postmanlabs/openapi-to-postman
- openapi-to-postmanv2 on npm — https://www.npmjs.com/package/openapi-to-postmanv2
- Insomnia: Import and export reference — https://developer.konghq.com/insomnia/import-export/
- Insomnia: Set a value from a response as an environment variable — https://developer.konghq.com/how-to/set-a-value-from-a-response-as-an-environment-variable/
- Prism GitHub Repository — https://github.com/stoplightio/prism
- @stoplight/prism-cli on npm — https://www.npmjs.com/package/@stoplight/prism-cli
- Twilio: Mock API Generation with Twilio's OpenAPI Spec — https://www.twilio.com/docs/openapi/mock-api-generation-with-twilio-openapi-spec
- Postman Environment Management (Tencent Cloud Article) — https://cloud.tencent.cn/developer/article/2533153
- Insomnia OpenAPI Import (NornicDB Docs) — https://github.com/orneryd/NornicDB/blob/main/docs/api-reference/openapi.md
- @apiaddicts/openapi2insomnia on npm — https://www.npmjs.com/package/@apiaddicts/openapi2insomnia
- @expediagroup/spec-transformer on npm — https://www.npmjs.com/package/@expediagroup/spec-transformer