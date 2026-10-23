# Laravel API Documentation: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

Laravel API documentation is the practice of systematically describing the endpoints, request/response formats, authentication requirements, and behavioural contracts of a Laravel-powered API so that consumers (frontend developers, mobile teams, third-party integrators) can understand and integrate with it without reading the source code.

**Technical Definition**

In the Laravel ecosystem, API documentation is typically expressed using the **OpenAPI Specification** (formerly Swagger Specification), a machine-readable standard (JSON or YAML) that describes RESTful APIs. Documentation generators such as **Scribe** and **L5-Swagger** extract metadata from Laravel routes, controllers, Form Requests, API Resources, and PHPDoc annotations to produce OpenAPI-compliant specifications, which are then rendered as interactive HTML documentation via tools like **Swagger UI** or **ReDoc**.

**Beginner-Friendly Explanation**

API documentation is the instruction manual for your API. It tells other developers what URLs they can call, what data to send, what they will get back, and how to authenticate. Without it, integrating with your API is like assembling furniture without instructions—frustrating and error-prone. Laravel provides tools that can generate this manual automatically from your code, so it stays up to date as your API evolves.

---

### Key Characteristics

- **Contract-First or Code-First**: Documentation can be written before the API (OpenAPI-first) or generated from existing Laravel code (code-first via Scribe, Scramble, or L5-Swagger).
- **OpenAPI as the Lingua Franca**: OpenAPI 3.x is the industry standard for describing REST APIs, enabling interoperability with Postman, Apifox, Redoc, Stoplight, and API gateways.
- **Multiple Tooling Options**: Scribe (annotation-light, multi-format export), L5-Swagger (strict OpenAPI annotations, Swagger UI), and Scramble (zero-annotation, static analysis) cater to different team preferences.
- **CI/CD Integration**: Documentation can be generated and deployed automatically on every merge, keeping docs in sync with code.
- **Interactive Exploration**: Swagger UI and Scribe's built-in tester allow consumers to try endpoints directly from the documentation.
- **Versioned Documentation**: API versioning (e.g., `/v1/`, `/v2/`) can be reflected in documentation, allowing multiple API generations to coexist.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** with API routes defined in `routes/api.php` (or versioned equivalents).
- **Composer** for installing documentation packages.
- **Basic knowledge of REST principles** (HTTP verbs, status codes, JSON payloads).
- **For Scribe**: PHP 7.4+ and Laravel/Lumen 6+.
- **For L5-Swagger**: Familiarity with PHPDoc annotations (`@OA\...`).

---

### Related Programming Areas

- **OpenAPI Specification**: The standard that documentation tools target.
- **API Versioning**: Documentation must reflect the version(s) of the API being described.
- **API Resources**: Laravel's `JsonResource` classes define response shapes that documentation tools can extract.
- **Form Requests**: Validation rules can be parsed to document request parameters.
- **CI/CD Pipelines**: Automated documentation generation fits naturally into deployment workflows.
- **API Testing**: Documentation and contract tests can be linked to ensure docs match runtime behaviour.

---

## Core Concepts

---

### 1. API Documentation

#### Definitions

**Core Definition**

API documentation is a structured description of an API's endpoints, parameters, request bodies, responses, authentication methods, and error codes, intended for developers who consume the API.

**Technical Definition**

API documentation in Laravel can be **hand-written** (Markdown files, static pages) or **auto-generated** from code metadata. Auto-generation tools parse routes, controller method signatures, Form Request validation rules, API Resource transformations, and PHPDoc annotations to produce a specification. The resulting specification is often expressed in **OpenAPI** format and rendered through a UI (Swagger UI, Redoc, or Scribe's HTML output).

**Beginner-Friendly Explanation**

API documentation is the user manual for your API. It tells consumers what they can do, what they need to send, and what they will receive.

#### Purposes

- To enable API consumers to integrate quickly and correctly.
- To serve as a contract between backend and frontend teams.
- To reduce support burden by answering common integration questions.
- To document authentication, rate limits, and error handling.
- To provide a reference for versioning and deprecation timelines.

#### Syntax Rules and Structure

**Hand-Written Documentation Structure**

```markdown
# API Documentation

## Authentication
All endpoints require a Bearer token in the `Authorization` header.

## Endpoints

### GET /api/v1/users
Returns a paginated list of users.

**Query Parameters:**
- `page` (integer, optional): Page number (default: 1)
- `per_page` (integer, optional): Items per page (default: 15)

**Response (200 OK):**
```json
{
  "data": [
    { "id": 1, "name": "John Doe", "email": "john@example.com" }
  ],
  "meta": { "current_page": 1, "total": 50 }
}
```
```

**Auto-Generated Documentation Structure (Scribe/L5-Swagger)**

Auto-generated docs derive structure from:
- Route definitions (`routes/api.php`)
- Controller method PHPDoc (`@bodyParam`, `@response`)
- Form Request `rules()` methods
- API Resource `toArray()` methods
- Model casts and attributes

**Syntax Rules**

- Each endpoint should document HTTP method, URI, authentication, parameters, request body (if applicable), and response examples.
- Error responses (4xx, 5xx) should be documented alongside success responses.
- Versioning should be explicit (e.g., `/api/v1/`).

**Constraints and Limitations**

- Hand-written docs drift out of sync with code unless disciplined review is enforced.
- Auto-generated docs may miss edge cases (dynamic responses, conditional middleware).
- Documentation cannot replace contract testing.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Scribe-Generated Documentation for a Simple Endpoint**

**Controller with Scribe Annotations**

```php
<?php
// app/Http/Controllers/Api/V1/UserController.php
namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Http\Resources\UserResource;
use App\Models\User;

class UserController extends Controller
{
    /**
     * List all users.
     *
     * @group User Management
     *
     * @queryParam page integer Page number. Example: 1
     * @queryParam per_page integer Items per page. Example: 15
     *
     * @response 200 {
     *   "data": [
     *     { "id": 1, "name": "John Doe", "email": "john@example.com" }
     *   ],
     *   "meta": { "current_page": 1, "total": 50 }
     * }
     */
    public function index()
    {
        return UserResource::collection(User::paginate(15));
    }
}
```

**Command**

```bash
php artisan scribe:generate
```

**Expected Output (generated HTML)**

The documentation includes a "User Management" group with a `GET /api/v1/users` endpoint, showing query parameters, a sample response, and a "Try It Out" tester.

**Why This Output Occurs**

Scribe parses the `@group` and `@queryParam` annotations, reads the controller return type, and generates an HTML page (and an OpenAPI spec) describing the endpoint.

---

**Example 2: L5-Swagger Annotations**

**Controller with OpenAPI Annotations**

```php
<?php
namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * @OA\Get(
     *     path="/api/v1/users",
     *     summary="List all users",
     *     tags={"Users"},
     *     security={{"bearerAuth":{}}},
     *     @OA\Parameter(
     *         name="page",
     *         in="query",
     *         description="Page number",
     *         required=false,
     *         @OA\Schema(type="integer", example=1)
     *     ),
     *     @OA\Response(
     *         response=200,
     *         description="Paginated list of users",
     *         @OA\JsonContent(
     *             type="object",
     *             @OA\Property(property="data", type="array", @OA\Items(ref="#/components/schemas/User"))
     *         )
     *     )
     * )
     */
    public function index()
    {
        // ...
    }
}
```

**Command**

```bash
php artisan l5-swagger:generate
```

**Expected Output**

Swagger UI renders the endpoint with parameters, security requirements, and response schema.

**Why This Output Occurs**

The `swagger-php` library parses the `@OA\Get` annotation and compiles it into an OpenAPI specification, which L5-Swagger serves via Swagger UI.

---

#### Real-World Cases with Explanation

**Case 1: SaaS Platform with External Integrators**

A SaaS platform exposes a public API to third-party developers. Auto-generated documentation (using Scribe or L5-Swagger) is published at `/docs` and kept in sync with the codebase via CI/CD, ensuring integrators always have accurate information.

**Explanation**: External consumers depend on accurate documentation; automation prevents drift.

**Case 2: Internal Frontend/Backend Contract**

A frontend team consumes a Laravel API built by a backend team. The API documentation serves as the contract, and frontend developers can use Swagger UI to test endpoints before writing integration code.

**Explanation**: Documentation acts as a shared source of truth between teams.

---

### 2. OpenAPI

#### Definitions

**Core Definition**

OpenAPI is a language-agnostic, machine-readable specification standard (JSON or YAML) for describing RESTful APIs, including endpoints, operations, parameters, request/response bodies, authentication schemes, and schemas.

**Technical Definition**

The OpenAPI Specification (OAS) is maintained by the OpenAPI Initiative (a Linux Foundation project). Version 3.x defines a structured document with `info`, `servers`, `paths`, `components`, `security`, and `tags` sections. Each path contains operations (GET, POST, etc.) with parameters, request bodies, and responses. Schemas are defined under `components/schemas` and referenced via `$ref`. Laravel tools like Scribe and L5-Swagger generate OAS 3.x documents.

**Beginner-Friendly Explanation**

OpenAPI is a standardized way to describe your API that both humans and computers can understand. It is like a blueprint that tools can read to generate documentation, client SDKs, and tests.

#### Purposes

- To provide a single, standardized contract for an API.
- To enable tooling interoperability (Swagger UI, Postman, Redoc, code generators).
- To support contract-first API design.
- To automate documentation rendering and validation.

#### Syntax Rules and Structure

**Complete General Syntax (OpenAPI 3.0 YAML)**

```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
  description: API for managing users.
servers:
  - url: https://api.example.com/v1
paths:
  /users:
    get:
      summary: List all users
      tags:
        - Users
      security:
        - bearerAuth: []
      parameters:
        - name: page
          in: query
          required: false
          schema:
            type: integer
            example: 1
      responses:
        '200':
          description: Paginated list of users
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `openapi` | Spec version (e.g., `3.0.0`). |
| `info` | API metadata (title, version, description). |
| `servers` | Base URLs for the API. |
| `paths` | Endpoint definitions and operations. |
| `components` | Reusable schemas, security schemes, parameters. |
| `security` | Global or per-operation authentication requirements. |
| `tags` | Grouping labels for endpoints. |

**Syntax Rules**

- OpenAPI documents can be JSON or YAML.
- `$ref` is used to reference reusable components.
- Every operation should have at least one documented response.
- Schema types must conform to JSON Schema conventions.

**Constraints and Limitations**

- OpenAPI describes REST APIs; it is less suited to GraphQL or gRPC.
- Complex conditional responses can be difficult to express.
- The specification is descriptive, not prescriptive; it does not enforce runtime behaviour.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Minimal OpenAPI Document**

**Code**

```yaml
openapi: 3.0.0
info:
  title: Ping API
  version: 1.0.0
paths:
  /ping:
    get:
      summary: Health check
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                type: object
                properties:
                  status:
                    type: string
                    example: ok
```

**Expected Output**

A valid OpenAPI 3.0 document that describes a single `GET /ping` endpoint returning `{"status": "ok"}`.

**Why This Output Occurs**

The document adheres to the OpenAPI 3.0 structure, defining the required `info`, `paths`, and `responses` sections.

---

**Example 2: OpenAPI with Reusable Components**

**Code**

```yaml
openapi: 3.0.0
info:
  title: Blog API
  version: 1.0.0
paths:
  /posts:
    get:
      summary: List posts
      responses:
        '200':
          description: A list of posts
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Post'
components:
  schemas:
    Post:
      type: object
      properties:
        id:
          type: integer
        title:
          type: string
        body:
          type: string
```

**Expected Output**

The `Post` schema is defined once and referenced wherever needed, reducing duplication.

**Why This Output Occurs**

`$ref` points to `#/components/schemas/Post`, enabling reuse across multiple endpoints.

---

#### Real-World Cases with Explanation

**Case 1: API Gateway Integration**

An organisation uses an API gateway (e.g., Kong, AWS API Gateway) that imports OpenAPI specifications to configure routing, rate limiting, and validation. The Laravel API generates an OpenAPI spec via Scribe/L5-Swagger, which is fed into the gateway.

**Explanation**: OpenAPI enables infrastructure tooling to understand and govern the API.

**Case 2: SDK Generation**

A platform generates client SDKs (TypeScript, Python, Java) from its OpenAPI specification, ensuring SDKs stay in sync with the API.

**Explanation**: OpenAPI's machine-readable nature makes it ideal for code generation.

---

### 3. Swagger

#### Definitions

**Core Definition**

Swagger is a suite of tools and a former name for the OpenAPI Specification. In modern usage, "Swagger" often refers to **Swagger UI**, a browser-based interface that renders OpenAPI specifications as interactive documentation.

**Technical Definition**

Swagger originally referred to the Swagger Specification, which was donated to the OpenAPI Initiative and renamed OpenAPI in 2016. The Swagger tooling ecosystem now includes **Swagger UI** (interactive documentation renderer), **Swagger Editor** (spec authoring tool), **Swagger Codegen** (SDK generator), and **SwaggerHub** (collaboration platform). In Laravel, Swagger UI is typically served via **L5-Swagger** (which wraps `swagger-php` and `swagger-ui`) or lightweight packages like `wotz/laravel-swagger-ui`.

**Beginner-Friendly Explanation**

Swagger is the tool that turns your OpenAPI specification into a beautiful, interactive webpage where you can read about endpoints and even try them out.

#### Purposes

- To render OpenAPI specifications as human-readable, interactive documentation.
- To provide a "Try It Out" interface for testing endpoints directly from the browser.
- To enable API exploration and debugging for consumers.
- To serve as a visual contract during development.

#### Syntax Rules and Structure

**L5-Swagger Installation and Configuration**

```bash
composer require darkaonline/l5-swagger
php artisan vendor:publish --provider "L5Swagger\L5SwaggerServiceProvider"
php artisan l5-swagger:generate
```

**Configuration (`config/l5-swagger.php`)**

```php
return [
    'default' => 'default',
    'documentations' => [
        'default' => [
            'api' => [
                'title' => 'Laravel API Documentation',
            ],
            'routes' => [
                'api' => 'api/documentation',
            ],
            'paths' => [
                'annotations' => [base_path('app')],
                'docs' => storage_path('api-docs'),
            ],
        ],
    ],
];
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `swagger-php` | Parses PHPDoc annotations into OpenAPI specs. |
| `swagger-ui` | Renders the spec as interactive HTML. |
| `l5-swagger:generate` | Compiles annotations into a spec file. |
| `/api/documentation` | Default route serving Swagger UI. |

**Syntax Rules**

- Annotations use `@OA\` prefixes (e.g., `@OA\Get`, `@OA\Post`).
- The `paths.annotations` configuration must include all directories containing annotations.
- Swagger UI requires a valid OpenAPI spec file (JSON or YAML).

**Constraints and Limitations**

- Annotations are verbose; a single endpoint can require 20–30 lines.
- Endpoints without annotations are omitted from the documentation.
- L5-Swagger is a wrapper; the actual specification authoring is done via `swagger-php`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: L5-Swagger Setup and Basic Annotation**

**Installation**

```bash
composer require darkaonline/l5-swagger
php artisan vendor:publish --provider "L5Swagger\L5SwaggerServiceProvider"
```

**Controller Annotation**

```php
<?php
namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;

class PingController extends Controller
{
    /**
     * @OA\Get(
     *     path="/api/v1/ping",
     *     summary="Health check",
     *     tags={"System"},
     *     @OA\Response(
     *         response=200,
     *         description="OK",
     *         @OA\JsonContent(
     *             type="object",
     *             @OA\Property(property="status", type="string", example="ok")
     *         )
     *     )
     * )
     */
    public function ping()
    {
        return response()->json(['status' => 'ok']);
    }
}
```

**Generate and View**

```bash
php artisan l5-swagger:generate
# Visit /api/documentation
```

**Expected Output**

Swagger UI displays the `GET /api/v1/ping` endpoint under the "System" tag, with a `200` response example `{"status": "ok"}`.

**Why This Output Occurs**

`swagger-php` parses the `@OA\Get` annotation and compiles it into an OpenAPI spec, which Swagger UI renders.

---

#### Real-World Cases with Explanation

**Case 1: Enterprise API with Strict Governance**

An enterprise team uses L5-Swagger because it enforces OpenAPI compliance and integrates with existing Swagger UI workflows, API gateways, and Postman collections. Annotations serve as the single source of truth for the API contract.

**Explanation**: Swagger tooling provides precise control and ecosystem compatibility for teams with formal API governance.

---

### 4. Scribe

#### Definitions

**Core Definition**

Scribe is a Laravel package that generates API documentation for humans from your Laravel/Lumen/Dingo codebase, producing a polished HTML page, an OpenAPI spec, and a Postman collection with minimal manual annotation.

**Technical Definition**

Scribe (`knuckleswtf/scribe`) scans Laravel routes, parses Form Request validation rules, inspects API Resource transformations, and reads PHPDoc annotations (`@bodyParam`, `@queryParam`, `@response`, `@group`) to produce documentation. It supports a "Try It Out" tester, code sample generation, and export to OpenAPI 3.0 and Postman formats. Scribe is designed to be annotation-light, inferring as much as possible from the codebase.

**Beginner-Friendly Explanation**

Scribe is a tool that reads your Laravel code and automatically writes API documentation for you. It needs very few annotations and produces a beautiful, interactive page.

#### Purposes

- To generate human-friendly API documentation with minimal annotation overhead.
- To produce OpenAPI and Postman artifacts alongside HTML docs.
- To keep documentation close to the code, reducing drift.
- To provide an interactive tester for API consumers.

#### Syntax Rules and Structure

**Installation**

```bash
composer require --dev knuckleswtf/scribe
php artisan vendor:publish --tag=scribe-config
```

**Configuration (`config/scribe.php`)**

```php
return [
    'title' => 'My API Documentation',
    'description' => 'Documentation for My API',
    'base_url' => env('APP_URL'),
    'type' => 'laravel',
    'routes' => [
        [
            'match' => ['prefixes' => ['api/*']],
            'include' => [],
            'exclude' => [],
        ],
    ],
];
```

**Annotation Syntax**

```php
/**
 * @group User Management
 *
 * @bodyParam name string required The user's name. Example: John Doe
 * @bodyParam email string required The user's email. Example: john@example.com
 *
 * @response 201 {
 *   "id": 1,
 *   "name": "John Doe",
 *   "email": "john@example.com"
 * }
 */
public function store(StoreUserRequest $request)
{
    // ...
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@group` | Groups endpoints in the documentation. |
| `@bodyParam` | Documents a request body parameter. |
| `@queryParam` | Documents a query parameter. |
| `@response` | Provides a response example. |
| `scribe:generate` | Generates the documentation. |

**Syntax Rules**

- Scribe can infer request parameters from Form Request rules and response shapes from API Resources.
- Annotations override inferred content when both exist.
- The generated documentation is available at `/docs` by default.
- OpenAPI and Postman exports are configured in `config/scribe.php`.

**Constraints and Limitations**

- Scribe relies on annotations for complete documentation; inference alone may miss edge cases.
- Laravel 11 requires careful strategy registration in `bootstrap/app.php` for some features (e.g., `@authenticated`).
- Scribe is a development dependency; it should not be installed in production.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Scribe Documentation for a Form Request**

**Form Request**

```php
<?php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
        ];
    }
}
```

**Controller**

```php
/**
 * @group User Management
 *
 * @response 201 {
 *   "id": 1,
 *   "name": "John Doe",
 *   "email": "john@example.com"
 * }
 */
public function store(StoreUserRequest $request)
{
    $user = User::create($request->validated());
    return new UserResource($user);
}
```

**Command**

```bash
php artisan scribe:generate
```

**Expected Output**

Scribe generates documentation listing `name`, `email`, and `password` as required body parameters (inferred from the Form Request), with a `201` response example from the `@response` annotation. The HTML page includes a "Try It Out" tester.

**Why This Output Occurs**

Scribe parses the `rules()` method of `StoreUserRequest` to infer request parameters and reads the `@response` annotation for the response example.

---

**Example 2: Generating OpenAPI and Postman Exports**

**Configuration (`config/scribe.php`)**

```php
'postman' => [
    'enabled' => true,
    'overrides' => [
        'info.title' => 'My API',
    ],
],
'openapi' => [
    'enabled' => true,
    'overrides' => [
        'info.version' => '1.0.0',
    ],
],
```

**Command**

```bash
php artisan scribe:generate --openapi
```

**Expected Output**

Alongside the HTML documentation, Scribe generates an OpenAPI 3.0 JSON/YAML file and a Postman collection.

**Why This Output Occurs**

The `openapi` and `postman` configuration blocks enable additional export formats during generation.

---

#### Real-World Cases with Explanation

**Case 1: Rapidly Iterating Startup**

A startup with a small team and frequent API changes uses Scribe because it requires minimal annotation overhead and generates usable docs automatically from Form Requests and API Resources. The team runs `scribe:generate` as part of their CI pipeline.

**Explanation**: Scribe's inference-first approach suits teams that prioritise speed over exhaustive annotation.

---

### 5. Laravel API Documentation

#### Definitions

**Core Definition**

Laravel API documentation refers to the practice and tooling ecosystem within Laravel for producing and maintaining API documentation, including Scribe, L5-Swagger, Scramble, and hand-written approaches.

**Technical Definition**

Laravel does not ship with a built-in API documentation generator. Instead, the ecosystem provides several packages: **Scribe** (inference + annotations), **L5-Swagger** (OpenAPI annotations + Swagger UI), **Scramble** (zero-annotation, static analysis), and **Laravel-apidoc-generator** (legacy). The choice depends on team preference, API complexity, and tooling integration requirements.

**Beginner-Friendly Explanation**

Laravel does not have one official documentation tool, but it has several excellent community packages. You pick the one that fits how your team works.

#### Purposes

- To provide a standardised approach to documenting Laravel APIs.
- To integrate documentation generation into Laravel's artisan command ecosystem.
- To leverage Laravel-specific features (Form Requests, API Resources) for documentation inference.
- To support multiple output formats (HTML, OpenAPI, Postman).

#### Syntax Rules and Structure

**Tool Comparison**

| Tool | Approach | Annotation Overhead | Output Formats | Best For |
|------|----------|---------------------|----------------|----------|
| **Scribe** | Inference + annotations | Low–Medium | HTML, OpenAPI, Postman | Fast-moving teams, mid-sized APIs |
| **L5-Swagger** | Strict OpenAPI annotations | High | Swagger UI, OpenAPI | Enterprise, OpenAPI-first workflows |
| **Scramble** | Zero-annotation static analysis | None | OpenAPI, HTML | Teams wanting automatic, code-as-docs |
| **Laravel-apidoc-generator** | Legacy annotation-based | Medium | HTML | Legacy projects (Laravel 5.7–10) |

**Syntax Rules**

- Scribe: `composer require --dev knuckleswtf/scribe`; generate with `php artisan scribe:generate`.
- L5-Swagger: `composer require darkaonline/l5-swagger`; generate with `php artisan l5-swagger:generate`.
- Scramble: `composer require dedoc/scramble`; generate with `php artisan scramble:export`.

**Constraints and Limitations**

- Scribe and L5-Swagger require annotations for complete documentation.
- Scramble may miss custom response construction or complex middleware logic.
- Laravel-apidoc-generator is not actively maintained for newer Laravel versions.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Choosing Between Scribe and L5-Swagger**

**Scribe Setup**

```bash
composer require --dev knuckleswtf/scribe
php artisan vendor:publish --tag=scribe-config
php artisan scribe:generate
```

**L5-Swagger Setup**

```bash
composer require darkaonline/l5-swagger
php artisan vendor:publish --provider "L5Swagger\L5SwaggerServiceProvider"
php artisan l5-swagger:generate
```

**Expected Output**

Both generate documentation, but Scribe produces HTML with an integrated tester, while L5-Swagger produces Swagger UI backed by an OpenAPI spec file.

**Why This Output Occurs**

Scribe and L5-Swagger use different architectures and output formats, reflecting their different design philosophies.

---

#### Real-World Cases with Explanation

**Case 1: Legacy Migration**

A team maintaining a Laravel 8 application uses Laravel-apidoc-generator because it supports their version. When they upgrade to Laravel 12, they migrate to Scribe or Scramble.

**Explanation**: Tool choice is often constrained by Laravel version and existing workflows.

---

### 6. API Documentation Best Practices

#### Definitions

**Core Definition**

API documentation best practices are the guidelines for producing documentation that is accurate, complete, maintainable, and genuinely useful to API consumers.

**Technical Definition**

Best practices encompass writing documentation as code, integrating generation into CI/CD, documenting authentication and errors, using consistent naming and examples, versioning documentation alongside the API, and validating documentation through contract tests.

**Beginner-Friendly Explanation**

Best practices are the habits that make documentation reliable and helpful rather than stale and misleading.

#### Key Best Practices

1. **Treat Documentation as Code**: Store annotations and configuration in version control. Generate docs in CI/CD so they deploy automatically with the API.

2. **Use OpenAPI as the Contract**: Whether using Scribe, L5-Swagger, or Scramble, produce an OpenAPI specification that can be consumed by downstream tools (Postman, gateways, SDK generators).

3. **Document Authentication and Authorization**: Clearly specify how to obtain and use tokens, and which scopes/abilities are required for each endpoint.

4. **Include Realistic Examples**: Provide request and response examples that reflect actual data, not placeholders like `"string"`.

5. **Document Error Responses**: Describe 4xx and 5xx responses, including error codes and messages.

6. **Version Your Documentation**: If your API has versions, produce versioned documentation (e.g., `/docs/v1`, `/docs/v2`) or clearly mark versioned endpoints.

7. **Validate Documentation with Contract Tests**: Use tools that compare the generated OpenAPI spec against actual API responses to detect drift.

8. **Choose the Right Tool for Your Team**: Scribe for speed, L5-Swagger for OpenAPI-first governance, Scramble for zero-annotation automation.

9. **Review Annotations in Pull Requests**: Make documentation completeness part of code review.

10. **Keep Docs Accessible**: Publish documentation at a stable URL (e.g., `/docs`) and protect internal docs behind authentication if necessary.

11. **Avoid Over-Documenting Internal Endpoints**: Focus public documentation on the public API surface; internal endpoints should not be exposed.

12. **Update Documentation with Deprecations**: When deprecating endpoints, document the deprecation, sunset date, and migration path.

#### Real-World Cases with Explanation

**Case 1: CI/CD-Integrated Documentation**

A team adds `php artisan scribe:generate` to their GitHub Actions workflow. On every merge to `main`, documentation is regenerated and deployed to `/docs`. Contract tests compare the generated OpenAPI spec against actual responses, failing the build if drift is detected.

**Explanation**: Automation ensures documentation never falls behind the code.

**Case 2: Multi-Version API Documentation**

A platform runs `/v1` and `/v2` APIs simultaneously. Scribe is configured with route groups for each version, producing separate documentation sections or separate specs, allowing consumers to choose the version they need.

**Explanation**: Versioned documentation supports gradual migrations and long-term API stability.

---

## References

- Scribe: Introduction — https://scribe.knuckles.wtf/laravel/3.x/
- Scribe: Installation and Configuration (Laravel Daily) — https://laraveldaily.com/lesson/api-laravel/generating-api-documentation-with-scribe
- L5-Swagger: OpenAPI/Swagger Integration for Laravel — https://github.com/darkaonline/l5-swagger
- Mastering API Versioning and Documentation in Laravel 12 — https://needlaravelsite.com/blog/mastering-api-versioning-and-documentation-in-laravel-12
- Zero-Config OpenAPI for Laravel: A Different Approach to API Documentation — https://blog.stackademic.com/zero-config-openapi-for-laravel-a-different-approach-to-api-documentation-f1877170a9d4
- Laravel API Documentation Tools Comparison (php.cn) — https://www.php.cn/faq/2637790.html
- JustSteveKing/kit: Opinionated Laravel API Starter Kit (Scribe + OpenAPI) — https://github.com/JustSteveKing/kit
- Laravel Swagger Integration: Step-by-Step Guide (Bagisto) — https://bagisto.com/en/laravel-swagger-integration-step-by-step-guide/
- Laravel Swagger UI Package (wotz/laravel-swagger-ui) — https://packagist.org/packages/wotz/laravel-swagger-ui
- Laravel API Response Builder with Auto-Generated Swagger Documentation — https://zenodo.org/records/10440753
- OpenAPI Specification (Official) — https://spec.openapis.org/oas/v3.0.3
- In-depth Guide on Documenting API Responses with Scramble (Laravel News) — https://laravel-news.com/scramble-documenting-api-responses

---

**Version Note**: This cheat sheet is based on Laravel 10.x through 13.x documentation and ecosystem packages as of 2025–2026. Scribe, L5-Swagger, and Scramble are actively maintained, but package versions and Laravel compatibility should be verified on Packagist before installation. Scramble's Pro features (e.g., advanced schema deduplication) may require a commercial licence. Developers should always consult the official documentation for their chosen tool and Laravel version.