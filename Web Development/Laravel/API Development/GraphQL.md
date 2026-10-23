# Laravel GraphQL & Lighthouse: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**

GraphQL is a query language and server-side runtime for APIs that allows clients to request exactly the data they need in a single request, using a strongly-typed schema as the contract between client and server. Lighthouse is a Laravel package that implements a GraphQL server by leveraging Laravel's Eloquent models, policies, and conventions.

**Technical Definition**

GraphQL defines a type system (schema) using the Schema Definition Language (SDL), and exposes three root operation types: `Query` (read), `Mutation` (write), and `Subscription` (real-time stream). Lighthouse (`nuwave/lighthouse`) compiles SDL schemas into executable GraphQL servers, using directives (e.g., `@paginate`, `@hasMany`, `@create`) to wire fields to Eloquent models and custom resolvers. It integrates with Laravel's service container, authentication guards, and event broadcasting system.

**Beginner-Friendly Explanation**

GraphQL is a smarter way to build APIs. Instead of calling many different endpoints that each return fixed data (like REST), you send one request that says exactly what you want, and the server sends back exactly that—nothing more, nothing less. Lighthouse is the tool that lets you build these GraphQL APIs inside Laravel, using the models and code you already have.

---

### Key Characteristics

- **Schema-First**: The schema is the source of truth; resolvers and directives attach behaviour to schema fields.
- **Client-Specified Responses**: Clients request only the fields they need, reducing over-fetching and under-fetching.
- **Single Endpoint**: All operations are served through a single HTTP endpoint (typically `/graphql`).
- **Directive-Driven**: Lighthouse provides dozens of built-in directives (`@all`, `@find`, `@paginate`, `@create`, `@update`, `@delete`, `@hasMany`, `@belongsTo`, `@guard`, `@rules`) that eliminate boilerplate.
- **Eloquent Integration**: Models map directly to GraphQL types; relationships are resolved with batched queries to prevent N+1.
- **Real-Time Subscriptions**: Built-in support for GraphQL subscriptions via Pusher, Laravel Echo, Redis, or Reverb.
- **Authentication and Authorization**: Leverages Laravel guards, `@guard`, and policies for field-level security.

---

### Prerequisites

- **PHP 8.1 or higher** (Laravel 10+) or **PHP 8.2** (Laravel 11+).
- **A Laravel application** with Eloquent models and a database.
- **Composer** for installing `nuwave/lighthouse`.
- **Basic knowledge of GraphQL concepts** (types, queries, mutations).
- **For subscriptions**: A broadcasting driver (Pusher, Redis, or Reverb) and a WebSocket client.

---

### Related Programming Areas

- **Eloquent**: Lighthouse resolves GraphQL fields using Eloquent models and relationships.
- **Authentication**: Laravel guards and Sanctum integrate with Lighthouse's `@guard` directive.
- **Broadcasting**: Subscriptions rely on Laravel's event broadcasting system.
- **Validation**: `@rules` and `@validator` directives reuse Laravel's validation rules.
- **Performance**: Batched loading and eager loading prevent the N+1 query problem.

---

## Core Concepts

---

### 1. GraphQL

#### Definitions

**Core Definition**

GraphQL is an open-source data query and manipulation language for APIs, and a runtime for fulfilling queries with existing data, developed by Facebook in 2012 and open-sourced in 2015.

**Technical Definition**

GraphQL defines a type system and an execution engine. A schema declares object types, scalar types, enums, interfaces, unions, and input types. Clients send documents containing operations (query, mutation, subscription) with selection sets. The server validates the document against the schema, executes resolvers for each field, and returns a JSON response shaped exactly like the selection set. Introspection allows clients to discover the schema at runtime.

**Beginner-Friendly Explanation**

GraphQL is a way to ask your API for exactly what you want. You write a "query" that looks a bit like JSON, and the server replies with a matching JSON object. No more guessing which endpoint gives you which data.

#### Purposes

- To eliminate over-fetching and under-fetching of data.
- To provide a single, strongly-typed contract for API consumers.
- To enable client-driven development, where frontends can evolve without backend changes.
- To support real-time data via subscriptions.
- To enable powerful developer tooling (introspection, GraphiQL, code generation).

#### Syntax Rules and Structure

**Complete General Syntax (GraphQL Document)**

```graphql
# Query
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    posts {
      title
    }
  }
}

# Mutation
mutation CreatePost($input: CreatePostInput!) {
  createPost(input: $input) {
    id
    title
  }
}

# Subscription
subscription OnPostUpdated($postId: ID!) {
  postUpdated(postId: $postId) {
    id
    title
  }
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `query` | Read operation. |
| `mutation` | Write operation. |
| `subscription` | Real-time operation. |
| `$variable` | Typed variable passed separately from the document. |
| Selection set | The fields requested in `{ ... }`. |

**Syntax Rules**

- Operations can be named or anonymous.
- Variables are declared with `$name: Type` and passed in a separate `variables` JSON object.
- Selection sets must only request fields that exist on the return type.
- Fragments (`fragment Name on Type { ... }`) enable reusable selection sets.

**Constraints and Limitations**

- GraphQL is not a replacement for REST in all cases; file uploads and caching require additional patterns.
- Complex authorization must be implemented at the field/resolver level.
- Deeply nested queries can be exploited for denial-of-service unless query depth/complexity limits are enforced.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: A Simple Query**

**Document**

```graphql
query {
  user(id: 1) {
    id
    name
    email
  }
}
```

**Expected Output**

```json
{
  "data": {
    "user": {
      "id": "1",
      "name": "John Doe",
      "email": "john@example.com"
    }
  }
}
```

**Why This Output Occurs**

The selection set requests only `id`, `name`, and `email`, so the server returns exactly those fields. The `id` is returned as a string because GraphQL's `ID` scalar serializes as a string.

---

**Example 2: A Mutation with Variables**

**Document**

```graphql
mutation CreatePost($title: String!, $body: String!) {
  createPost(title: $title, body: $body) {
    id
    title
  }
}
```

**Variables**

```json
{ "title": "Hello GraphQL", "body": "This is my first post." }
```

**Expected Output**

```json
{
  "data": {
    "createPost": {
      "id": "1",
      "title": "Hello GraphQL"
    }
  }
}
```

**Why This Output Occurs**

The mutation creates a post and returns the requested `id` and `title` fields. Variables allow the same document to be reused with different values.

---

#### Real-World Cases with Explanation

**Case 1: Mobile App with Variable Bandwidth**

A mobile app needs user profiles but only has limited bandwidth. Using GraphQL, the app requests only `name` and `avatarUrl`, avoiding the large `bio` and `posts` fields that a REST endpoint might return.

**Explanation**: GraphQL's client-specified selection sets reduce payload size, which is critical for mobile performance.

---

### 2. Lighthouse

#### Definitions

**Core Definition**

Lighthouse is a PHP package that provides a GraphQL server implementation for Laravel, using a schema-first approach and Eloquent integration.

**Technical Definition**

Lighthouse (`nuwave/lighthouse`) compiles `.graphql` schema files into a Webonyx GraphQL server. It provides directives that resolve fields using Eloquent models (`@all`, `@find`, `@paginate`), perform CRUD operations (`@create`, `@update`, `@delete`), define relationships (`@hasMany`, `@belongsTo`), enforce authentication (`@guard`), and validate input (`@rules`). Custom resolvers are placed in `App\GraphQL\Queries`, `App\GraphQL\Mutations`, and `App\GraphQL\Subscriptions`.

**Beginner-Friendly Explanation**

Lighthouse is the Laravel package that makes it easy to build GraphQL APIs. You write a schema file, add some special directives, and Lighthouse connects everything to your database automatically.

#### Purposes

- To provide a first-class GraphQL server for Laravel with minimal boilerplate.
- To leverage existing Eloquent models and relationships.
- To offer built-in directives for common API operations.
- To integrate seamlessly with Laravel's authentication, validation, and broadcasting.

#### Syntax Rules and Structure

**Installation**

```bash
composer require nuwave/lighthouse
php artisan vendor:publish --tag=lighthouse-schema
php artisan vendor:publish --tag=lighthouse-config
```

**Configuration (`config/lighthouse.php`)**

```php
'route' => [
    'uri' => '/graphql',
    'middleware' => [
        \Nuwave\Lighthouse\Http\Middleware\AttemptAuthentication::class,
    ],
],
'schema_path' => base_path('graphql/schema.graphql'),
'namespaces' => [
    'models' => ['App', 'App\\Models'],
    'queries' => 'App\\GraphQL\\Queries',
    'mutations' => 'App\\GraphQL\\Mutations',
    'subscriptions' => 'App\\GraphQL\\Subscriptions',
],
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `schema_path` | Path to the root schema file. |
| `route.uri` | GraphQL endpoint (default `/graphql`). |
| `namespaces` | Auto-discovery namespaces for resolvers. |
| `AttemptAuthentication` | Attempts to authenticate on each request. |

**Syntax Rules**

- The root schema file is `graphql/schema.graphql` by default.
- Resolver classes are auto-discovered in the configured namespaces.
- The `lighthouse:query`, `lighthouse:mutation`, and `lighthouse:subscription` commands generate resolver classes.

**Constraints and Limitations**

- Lighthouse is tightly coupled to Eloquent; non-Eloquent data sources require custom resolvers.
- Schema must be compiled and cached in production (`lighthouse:cache`).

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Generating a Query Resolver**

**Command**

```bash
php artisan lighthouse:query Hello
```

**Generated Class**

```php
<?php
// app/GraphQL/Queries/Hello.php
namespace App\GraphQL\Queries;

final class Hello
{
    public function __invoke(): string
    {
        return 'world!';
    }
}
```

**Schema**

```graphql
type Query {
  hello: String! @field(resolver: "App\\GraphQL\\Queries\\Hello")
}
```

**Expected Output (Query)**

```graphql
{ hello }
```

```json
{ "data": { "hello": "world!" } }
```

**Why This Output Occurs**

The `@field` directive points to the `Hello` resolver class, whose `__invoke` method returns `"world!"`. Lighthouse calls this method when the `hello` field is queried.

---

#### Real-World Cases with Explanation

**Case 1: Rapid API Development**

A team building a SaaS backend uses Lighthouse to expose their Eloquent models as a GraphQL API. They define the schema once and use built-in directives (`@all`, `@paginate`, `@create`) for standard CRUD, writing custom resolvers only for business-specific logic.

**Explanation**: Lighthouse's directive-driven approach dramatically reduces the code required to build a full-featured GraphQL API.

---

### 3. GraphQL Schema

#### Definitions

**Core Definition**

A GraphQL schema is the strongly-typed contract that defines all types, fields, operations, and relationships available in a GraphQL API.

**Technical Definition**

The schema is written in SDL and includes object types (`type User { ... }`), scalar types (`String`, `Int`, `ID`, `DateTime`), enums, interfaces, unions, and input types. It defines the root `Query`, `Mutation`, and `Subscription` types, which are the entry points for operations. Lighthouse extends SDL with directives that attach resolvers and behaviour to fields.

**Beginner-Friendly Explanation**

The schema is the blueprint of your API. It tells clients what data exists, what they can ask for, and what they can change.

#### Purposes

- To define the shape of all data and operations in the API.
- To serve as a contract between frontend and backend teams.
- To enable validation, introspection, and tooling.
- To organize types and relationships clearly.

#### Syntax Rules and Structure

**Complete General Syntax**

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]! @hasMany
}

type Post {
  id: ID!
  title: String!
  body: String!
  author: User! @belongsTo
}

type Query {
  users: [User!]! @paginate
  user(id: ID! @whereKey): User @find
  posts: [Post!]! @paginate
}

type Mutation {
  createPost(input: CreatePostInput! @spread): Post! @create
  updatePost(id: ID!, input: UpdatePostInput! @spread): Post @update
  deletePost(id: ID!): Post @delete
}

input CreatePostInput {
  title: String! @rules(apply: ["required", "max:255"])
  body: String! @rules(apply: ["required"])
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `type` | Object type definition. |
| `!` | Non-null modifier. |
| `[Type!]!` | Non-null list of non-null items. |
| `@hasMany`, `@belongsTo` | Relationship directives. |
| `@paginate`, `@find`, `@create` | Eloquent operation directives. |
| `input` | Input type for mutations. |

**Syntax Rules**

- Every field must have a type.
- The `Query` type is mandatory; `Mutation` and `Subscription` are optional.
- Directives begin with `@` and are the primary way to add behaviour.
- Schemas can be split across files using `#import` and `extend type`.

**Constraints and Limitations**

- Schema changes can break existing clients; versioning or deprecation is required.
- Deeply nested schemas can lead to performance issues without batched loading.
- Custom scalar types must be registered and implemented.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: A Complete Schema with Relationships**

**Schema**

```graphql
type User {
  id: ID!
  name: String!
  posts: [Post!]! @hasMany
}

type Post {
  id: ID!
  title: String!
  author: User! @belongsTo
}

type Query {
  users: [User!]! @paginate(defaultCount: 10)
  user(id: ID! @whereKey): User @find
  posts: [Post!]! @paginate(defaultCount: 20)
}
```

**Query**

```graphql
{
  users(first: 5) {
    data {
      name
      posts {
        title
      }
    }
  }
}
```

**Expected Output**

A paginated list of users, each with their associated post titles, resolved in batched queries.

**Why This Output Occurs**

`@hasMany` and `@belongsTo` inform Lighthouse of the relationships, enabling it to batch-load related models and prevent N+1 queries.

---

**Example 2: Schema Organisation with Imports**

**Files**

```graphql
# graphql/schema.graphql
#import user.graphql
#import blog/*.graphql

type Query
```

```graphql
# graphql/user.graphql
type User {
  id: ID!
  name: String!
}

extend type Query {
  users: [User!]! @paginate
}
```

**Expected Output**

The final schema merges the imported types and extended `Query` fields into a single schema.

**Why This Output Occurs**

`#import` pastes the contents of imported files, and `extend type Query` adds fields to the root `Query` type.

---

#### Real-World Cases with Explanation

**Case 1: E-commerce Schema**

An e-commerce API defines `User`, `Product`, `Order`, and `OrderItem` types. Relationships (`@hasMany`, `@belongsTo`) connect them, and queries (`@paginate`, `@find`) expose paginated lists and single-record lookups.

**Explanation**: A well-organized schema makes the API intuitive and enables efficient batched data loading.

---

### 4. GraphQL Queries

#### Definitions

**Core Definition**

GraphQL queries are read-only operations that request specific fields from the schema, allowing clients to fetch exactly the data they need.

**Technical Definition**

Queries are defined as fields on the root `Query` type. Lighthouse resolves these fields using directives like `@all` (all records), `@find` (single record by primary key), `@paginate` (paginated list), `@first` (first matching record), and constraint directives like `@eq`, `@where`, `@orderBy`, and `@scope`. Custom queries use resolver classes in `App\GraphQL\Queries`.

**Beginner-Friendly Explanation**

Queries are how you ask for data. You specify which fields you want, and the server returns them.

#### Purposes

- To retrieve data from the API with client-specified field selection.
- To support filtering, sorting, and pagination without custom code.
- To enable efficient, batched data fetching through relationships.
- To provide a flexible read API that adapts to client needs.

#### Syntax Rules and Structure

**Complete General Syntax**

```graphql
type Query {
  users: [User!]! @all
  user(id: ID! @eq): User @find
  posts: [Post!]! @paginate(defaultCount: 10)
  publishedPosts: [Post!]! @all @where(operator: "=") ... # with scopes
  searchPosts(title: String @like(template: "%{}%")): [Post!]! @all
}
```

**Component Breakdown**

| Directive | Description |
|-----------|-------------|
| `@all` | Retrieve all records. |
| `@find` | Retrieve a single record by primary key. |
| `@first` | Retrieve the first matching record. |
| `@paginate` | Paginate results. |
| `@eq` | Exact-match constraint on an argument. |
| `@where` | Custom `WHERE` clause. |
| `@orderBy` | Sort results. |

**Syntax Rules**

- Arguments can be decorated with constraint directives.
- `@paginate` transforms the return type into a paginator type with `data` and `paginatorInfo`.
- Custom resolvers are auto-discovered in `App\GraphQL\Queries`.

**Constraints and Limitations**

- Unbounded `@all` queries can be dangerous on large tables; use pagination.
- Complex filters may require custom resolvers or the `@whereConditions` directive.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Paginated Query**

**Schema**

```graphql
type Query {
  posts: [Post!]! @paginate(defaultCount: 10)
}
```

**Query**

```graphql
{
  posts(first: 5) {
    data {
      id
      title
    }
    paginatorInfo {
      currentPage
      lastPage
    }
  }
}
```

**Expected Output**

```json
{
  "data": {
    "posts": {
      "data": [
        { "id": "1", "title": "First Post" }
      ],
      "paginatorInfo": {
        "currentPage": 1,
        "lastPage": 5
      }
    }
  }
}
```

**Why This Output Occurs**

`@paginate` transforms the field into a paginator, exposing `data` and `paginatorInfo` fields. The `first` argument controls page size.

---

**Example 2: Filtered Query with Constraints**

**Schema**

```graphql
type Query {
  posts(
    published: Boolean @eq
    title: String @like(template: "%{}%")
  ): [Post!]! @paginate
}
```

**Query**

```graphql
{
  posts(published: true, title: "GraphQL") {
    data { title }
  }
}
```

**Expected Output**

Only published posts whose title contains "GraphQL" are returned.

**Why This Output Occurs**

`@eq` applies an exact-match constraint on `published`, and `@like` applies a `LIKE` constraint on `title`.

---

#### Real-World Cases with Explanation

**Case 1: Dashboard Feed**

A dashboard needs the 10 most recent published posts with their authors. A paginated query with `@orderBy(column: "created_at", order: DESC)` and `@hasMany`/`@belongsTo` relationships fetches everything in a small number of batched queries.

**Explanation**: GraphQL queries with pagination, ordering, and relationship directives provide efficient, flexible data fetching.

---

### 5. GraphQL Mutations

#### Definitions

**Core Definition**

GraphQL mutations are write operations that create, update, or delete data on the server.

**Technical Definition**

Mutations are fields on the root `Mutation` type. Lighthouse provides `@create`, `@update`, `@delete`, `@upsert`, `@restore`, and `@forceDelete` directives that perform Eloquent operations. Input types are defined with `input` and validated using `@rules` or `@validator`. Mutations can be guarded with `@guard` and can inject context (e.g., the authenticated user's ID) using `@inject`.

**Beginner-Friendly Explanation**

Mutations are how you change data. You send a request that says "create this," "update that," or "delete this," and the server performs the action and returns the result.

#### Purposes

- To create, update, and delete records via the API.
- To validate input using Laravel's validation rules.
- To enforce authentication and authorization on write operations.
- To return the resulting data in the same request.

#### Syntax Rules and Structure

**Complete General Syntax**

```graphql
type Mutation {
  createPost(input: CreatePostInput! @spread): Post! @create
  updatePost(id: ID!, input: UpdatePostInput! @spread): Post @update
  deletePost(id: ID!): Post @delete
  upsertPost(input: UpsertPostInput! @spread): Post! @upsert
}

input CreatePostInput {
  title: String! @rules(apply: ["required", "max:255"])
  body: String! @rules(apply: ["required"])
}
```

**Component Breakdown**

| Directive | Description |
|-----------|-------------|
| `@create` | Create a new model. |
| `@update` | Update an existing model. |
| `@delete` | Delete a model. |
| `@upsert` | Create or update a model. |
| `@spread` | Spread input object fields as arguments. |
| `@rules` | Apply Laravel validation rules. |
| `@guard` | Require authentication. |
| `@inject` | Inject context values (e.g., `user.id`). |

**Syntax Rules**

- Input types are declared with `input`.
- `@spread` flattens the input object into individual arguments.
- `@create` and `@update` require the model to be inferable from the return type.
- `@guard` can be applied to a single field or an entire type.

**Constraints and Limitations**

- Mutations that affect multiple models require custom resolvers.
- Mass assignment protection must be configured on Eloquent models.
- Input validation should be applied consistently to prevent invalid data.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Create Mutation with Validation**

**Schema**

```graphql
type Mutation {
  createPost(input: CreatePostInput! @spread): Post! @create @guard
}

input CreatePostInput {
  title: String! @rules(apply: ["required", "max:255"])
  body: String! @rules(apply: ["required"])
}
```

**Mutation**

```graphql
mutation {
  createPost(title: "Hello", body: "GraphQL is great") {
    id
    title
  }
}
```

**Expected Output**

```json
{
  "data": {
    "createPost": {
      "id": "1",
      "title": "Hello"
    }
  }
}
```

**Why This Output Occurs**

`@create` creates a new `Post` model with the provided arguments. `@rules` validates the input. `@guard` ensures the user is authenticated before the mutation executes.

---

**Example 2: Update Mutation with Spread Input**

**Schema**

```graphql
type Mutation {
  updatePost(id: ID!, input: UpdatePostInput! @spread): Post @update
}

input UpdatePostInput {
  title: String @rules(apply: ["max:255"])
  body: String
}
```

**Mutation**

```graphql
mutation {
  updatePost(id: 1, title: "Updated Title") {
    id
    title
  }
}
```

**Expected Output**

```json
{
  "data": {
    "updatePost": {
      "id": "1",
      "title": "Updated Title"
    }
  }
}
```

**Why This Output Occurs**

`@update` finds the post by `id` and updates the provided fields. `@spread` allows the input fields to be passed as individual arguments.

---

#### Real-World Cases with Explanation

**Case 1: User Registration**

A registration mutation creates a `User` with validated `name`, `email`, and `password` inputs. The `@rules` directive enforces `email` uniqueness and password complexity.

**Explanation**: Mutations with validation and authentication directives provide a secure, server-side write API.

---

### 6. GraphQL Subscriptions

#### Definitions

**Core Definition**

GraphQL subscriptions are long-lived operations that push real-time updates to clients when specific events occur on the server.

**Technical Definition**

Subscriptions are fields on the root `Subscription` type. Lighthouse registers subscribers and broadcasts updates via a broadcaster (Pusher by default, or Laravel Echo, Redis, or Reverb). Subscription classes extend `Nuwave\Lighthouse\Schema\Types\GraphQLSubscription` and implement a `filter` method to determine which subscribers receive each update. The `@broadcast` directive triggers a subscription from a mutation or model event.

**Beginner-Friendly Explanation**

Subscriptions let your app receive live updates. For example, when a new message is posted, everyone subscribed to that conversation gets it instantly, without refreshing the page.

#### Purposes

- To deliver real-time updates without polling.
- To support chat, notifications, live feeds, and collaborative features.
- To filter updates so subscribers only receive relevant events.
- To integrate with Laravel's event broadcasting ecosystem.

#### Syntax Rules and Structure

**Complete General Syntax**

```graphql
type Subscription {
  postUpdated(postId: ID!): Post @subscription(class: "App\\GraphQL\\Subscriptions\\PostUpdated")
}
```

**Subscription Class**

```php
<?php
namespace App\GraphQL\Subscriptions;

use Nuwave\Lighthouse\Schema\Types\GraphQLSubscription;
use Nuwave\Lighthouse\Subscriptions\Subscriber;

final class PostUpdated extends GraphQLSubscription
{
    public function filter(Subscriber $subscriber, mixed $root): bool
    {
        return $root->id == $subscriber->args['postId'];
    }
}
```

**Configuration (`config/lighthouse.php`)**

```php
'subscriptions' => [
    'broadcaster' => env('LIGHTHOUSE_BROADCASTER', 'pusher'),
    'version' => 2, // Recommended for new applications
],
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `@subscription` | Links the field to a subscription class. |
| `filter()` | Determines whether a subscriber receives the update. |
| `$subscriber->args` | Arguments the client used when subscribing. |
| `LIGHTHOUSE_BROADCASTER` | Broadcaster driver (`pusher`, `echo`, `redis`, `reverb`). |

**Syntax Rules**

- Subscription classes must extend `GraphQLSubscription`.
- The `filter` method returns `true` to send the update, `false` to skip.
- `subscriptions.version` controls the response format; version 2 is recommended.
- The broadcaster is configured via `LIGHTHOUSE_BROADCASTER` and the `subscriptions` config block.

**Constraints and Limitations**

- Subscriptions require a WebSocket-capable broadcaster.
- Pusher is the default; Redis/Echo and Reverb require additional packages or configuration.
- Filtering logic runs on the server for every subscriber, which can impact performance at scale.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Filtered Subscription**

**Schema**

```graphql
type Subscription {
  postUpdated(postId: ID!): Post @subscription(class: "App\\GraphQL\\Subscriptions\\PostUpdated")
}
```

**Subscription Class**

```php
final class PostUpdated extends GraphQLSubscription
{
    public function filter(Subscriber $subscriber, mixed $root): bool
    {
        return $root->id == $subscriber->args['postId'];
    }
}
```

**Client Subscription**

```graphql
subscription {
  postUpdated(postId: 1) {
    id
    title
  }
}
```

**Expected Output**

The client receives an update only when the post with ID `1` is updated.

**Why This Output Occurs**

The `filter` method compares the updated post's ID with the `postId` argument provided by the subscriber, returning `true` only for matching subscribers.

---

**Example 2: "Only to Others" Filtering**

**Subscription Class**

```php
public function filter(Subscriber $subscriber, mixed $root): bool
{
    return $subscriber->socket_id !== request()->header('X-Socket-ID');
}
```

**Expected Output**

The subscriber who triggered the update (identified by `X-Socket-ID`) does not receive their own update, while other subscribers do.

**Why This Output Occurs**

The filter compares the subscriber's socket ID with the `X-Socket-ID` header, implementing Laravel's "to others" broadcasting behaviour.

---

#### Real-World Cases with Explanation

**Case 1: Live Chat**

A chat application uses a `messageAdded` subscription filtered by `conversationId`. When a message is sent, only participants in that conversation receive the update in real time.

**Explanation**: Filtered subscriptions prevent irrelevant updates from being broadcast to all clients.

---

### 7. GraphQL Best Practices

#### Definitions

**Core Definition**

GraphQL best practices are the guidelines for designing secure, performant, and maintainable GraphQL APIs with Lighthouse.

**Technical Definition**

Best practices encompass schema design (nullable fields, pagination, naming conventions), performance (batched loading, query complexity limits), security (authentication, authorization, depth limiting), and operational concerns (caching, monitoring, versioning).

**Beginner-Friendly Explanation**

Best practices are the habits that keep your GraphQL API fast, safe, and easy to use.

#### Key Best Practices

1. **Use Batched Loading to Prevent N+1**: Use `@hasMany`, `@belongsTo`, and `@with` directives so Lighthouse can batch relationship queries. For non-Eloquent relationships, implement custom batch loaders.

2. **Paginate Lists**: Never expose unbounded `@all` queries on large tables. Use `@paginate` with `defaultCount` and `maxCount`.

3. **Validate All Input**: Apply `@rules` or `@validator` to every input field. Reuse Laravel validation rules rather than duplicating logic.

4. **Guard Sensitive Fields**: Use `@guard` on mutations and private queries. Configure the correct guard (e.g., `sanctum`) in `lighthouse.php`.

5. **Limit Query Depth and Complexity**: Configure `max_query_complexity` and `max_query_depth` in `lighthouse.php` to prevent denial-of-service attacks.

6. **Use Schema Imports and Type Extensions**: Organize large schemas into multiple files with `#import` and `extend type`.

7. **Name Primary Keys `id`**: Follow the convention of naming the primary key field `id` for client-side caching compatibility.

8. **Cache the Schema in Production**: Run `php artisan lighthouse:cache` during deployment to avoid compiling the schema on every request.

9. **Version Your API**: Use URL versioning (`/graphql/v1`) or schema directives to manage breaking changes and deprecations.

10. **Monitor Performance**: Use Laravel Telescope or dedicated APM tools to track query counts and resolver performance. Log slow queries and N+1 violations.

11. **Configure Subscriptions Carefully**: Use version 2 of the subscriptions protocol and choose the appropriate broadcaster for your scale (Pusher, Redis, Reverb).

12. **Test Resolvers and Schema**: Write tests for custom resolvers, mutations, and subscription filters. Validate the schema with `php artisan lighthouse:validate-schema`.

#### Multiple Annotated Complete Step by Step Code Examples and Expected Outputs

**Example 1: Pagination and Guarding**

**Schema**

```graphql
type Query {
  posts: [Post!]! @paginate(defaultCount: 10, maxCount: 50)
  me: User @auth
}

type Mutation {
  createPost(input: CreatePostInput! @spread): Post! @create @guard
}
```

**Expected Output**

- `posts` returns at most 50 items per page, with a default of 10.
- `me` requires authentication.
- `createPost` requires authentication.

**Why This Output Occurs**

`@paginate` enforces pagination limits. `@auth` and `@guard` enforce authentication at the field level.

---

**Example 2: Schema Caching in Production**

**Command**

```bash
php artisan lighthouse:cache
```

**Expected Output**

The compiled schema is cached, reducing request latency.

**Why This Output Occurs**

Lighthouse compiles the SDL schema into an executable form; caching avoids repeated compilation.

---

#### Real-World Cases with Explanation

**Case 1: Public API with Security and Performance**

A public GraphQL API uses pagination on all list fields, `@guard` on mutations, query depth limiting, and batched loading. The schema is organized into modular files and cached in production. These practices prevent abuse, ensure fast responses, and keep the codebase maintainable.

**Explanation**: Combining performance, security, and organisational best practices results in a production-ready GraphQL API.

---

## References

- Lighthouse Artisan Commands — https://lighthouse-php.com/3/api-reference/commands.html
- How to Build GraphQL APIs with Laravel Lighthouse (OneUptime) — https://oneuptime.com/blog/post/2026-01-27-laravel-lighthouse-graphql/view
- Lighthouse Fields (Resolvers) — https://lighthouse-php.com/6/the-basics/fields.html
- Filtering Subscriptions — https://lighthouse-php.com/master/subscriptions/filtering-subscriptions.html
- The N+1 Query Problem — https://lighthouse-php.com/master/performance/n-plus-one.html
- Authentication — https://lighthouse-php.com/6/security/authentication.html
- Directives — https://lighthouse-php.com/5/the-basics/directives.html
- Eloquent: Getting Started (Pagination, Constraints) — https://lighthouse-php.com/5/eloquent/getting-started.html
- Schema Organisation — https://lighthouse-php.com/master/digging-deeper/schema-organisation.html
- Subscriptions: Getting Started (versions, broadcasters) — https://raw.githubusercontent.com/nuwave/lighthouse/master/docs/master/subscriptions/getting-started.md
- GraphQL Official Specification — https://spec.graphql.org/
- Lighthouse GitHub Repository — https://github.com/nuwave/lighthouse

---

**Version Note**: This cheat sheet is based on Lighthouse 5.x and 6.x documentation. Subscription version 2 is recommended for new applications. Laravel Reverb compatibility requires Lighthouse 6.47.0 or later. Schema directives and auto-discovery namespaces may vary between major versions. Developers should always consult the official Lighthouse documentation for their specific version and Laravel release.