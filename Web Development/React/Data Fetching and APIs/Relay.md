# Relay — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Relay is a GraphQL client for React that enforces colocated, composable, and type-safe data fetching, where each component declares exactly what data it needs via GraphQL fragments, and Relay handles fetching, caching, consistency, and updates.

**Technical Definition:** Relay is an ahead-of-time compiled GraphQL client built by Meta. Its core architecture rests on five principles: **co-location** (each component declares its own data requirements via a GraphQL fragment adjacent to its rendering code), **data masking** (a component can only access fields it explicitly selected in its own fragment; parents cannot see child fragment data and vice versa), **composition** (fragments compose into parent fragments and ultimately into queries, forming a tree that mirrors the component tree), **render-as-you-fetch** (data fetching begins before the component that needs it renders, eliminating request waterfalls), and a **normalized store** (a flat, ID-keyed in-memory cache where every entity is stored once and any component reading updated data re-renders automatically). The Relay Compiler processes all GraphQL operations at build time, validating them against the schema and generating runtime artifacts, Flow/TypeScript types, and optimized queries. The Hooks-based APIs (`usePreloadedQuery`, `useFragment`, `usePaginationFragment`, `useMutation`, `useSubscription`) take advantage of compiler features to automate error-prone tasks such as generating refetch and pagination queries.

**Beginner-Friendly Explanation:** Relay is a GraphQL client that takes the idea of "each component asks for exactly what it needs" very seriously. You write a small GraphQL snippet (a fragment) inside each component, describing the fields that component uses. Relay's compiler reads all these snippets at build time, combines them into efficient queries, and generates TypeScript types for you. At runtime, Relay fetches the data in one request, stores it in a smart cache that keeps each item once, and automatically updates every component that uses the changed data. It is more opinionated and requires more upfront setup than Apollo Client, but it scales exceptionally well for large applications because the data requirements are always colocated with the UI that uses them.

### Key Characteristics

- **Co-location:** Each component declares its data requirements via a fragment right next to its rendering code.
- **Data masking:** Components can only access fields they explicitly selected; implicit coupling between components is prevented.
- **Composition:** Fragments compose into a tree that mirrors the component tree; the compiler flattens it into one network request per query.
- **Render-as-you-fetch:** `loadQuery` + `usePreloadedQuery` start fetching before the component renders, avoiding waterfalls.
- **Normalized store:** A flat, ID-keyed cache; every entity is stored once, and all readers re-render on updates.
- **Ahead-of-time compilation:** The Relay Compiler validates operations at build time and generates runtime artifacts and types.
- **Hooks-first API:** `usePreloadedQuery`, `useFragment`, `usePaginationFragment`, `useRefetchableFragment`, `useMutation`, `useSubscription`.
- **Compiler directives:** `@connection`, `@refetchable`, `@argumentDefinitions`, `@arguments`, `@relay(plural:)`, `@required`, `@inline` control code generation and runtime behaviour.
- **Connections:** Cursor-based pagination via the Connection abstraction (`edges`, `node`, `pageInfo`).
- **Updaters:** Imperative store mutations for complex client updates, with optimistic and regular variants.

### Prerequisites

- React function components, `useState`, and `useEffect`.
- GraphQL fundamentals: queries, mutations, fragments, schemas, and the Connection specification.
- JavaScript Promises and `async`/`await`.
- TypeScript or Flow (Relay generates typed artifacts; TypeScript is recommended).
- A GraphQL server with a Relay-compatible schema (Node interface, Connections).

### Related Programming Areas

- **GraphQL** — the query language and schema.
- **State Management** — normalized stores, local state, and cache policies.
- **Data Fetching** — render-as-you-fetch, Suspense, and loading states.
- **Performance** — compiler optimization, fragment granularity, and `@defer`/`@stream`.
- **Pagination** — the Connection specification and cursor-based pagination.

---

## 1. Relay & Relay Compiler

### Definitions

**Core Definition:** Relay is the runtime library (`react-relay`, `relay-runtime`); the Relay Compiler is the build-time tool that processes GraphQL operations and generates artifacts and types.

**Technical Definition:** Relay's ahead-of-time compilation requires the Relay Compiler, installed as `relay-compiler`. The compiler discovers all `graphql` tagged templates, validates them against the schema, applies Relay directives, generates optimized query text and runtime artifacts (`.graphql.js` files), and produces Flow or TypeScript types. Configuration lives in `relay.config.js`, `relay.config.json`, or a `"relay"` key in `package.json`. The compiler runs in watch mode during development and in CI for validation.

**Beginner-Friendly Explanation:** Relay has two halves: the compiler, which runs when you build your app and turns your GraphQL snippets into optimized code and TypeScript types; and the runtime, which fetches data and manages the cache in the browser.

### Purposes

- To validate all GraphQL operations against the schema at build time.
- To generate optimized query text and runtime artifacts.
- To generate Flow or TypeScript types for operations and fragments.
- To apply Relay directives (`@connection`, `@refetchable`, etc.) during compilation.
- To enable ahead-of-time optimization and persisted queries.

### Syntax Rules and Structure

**Installation:**

```bash
yarn add --dev relay-compiler
```

**Configuration (`relay.config.js`):**

```js
module.exports = {
  src: './src',
  language: 'typescript',
  schema: './schema.graphql',
  excludes: ['**/node_modules/**', '**/__mocks__/**', '**/__generated__/**'],
};
```

**Compiler config locations:** `relay.config.json`, `relay.config.(js/mjs/ts)`, or `"relay"` in `package.json`.

**Syntax Rules:**

1. The compiler requires a schema file (`schema.graphql`) or schema endpoint.
2. `src` points to the source directory containing `graphql` tagged templates.
3. `language` is `typescript` or `flow`.
4. Generated artifacts are written next to the source files by default.
5. Run the compiler in watch mode during development and in CI for validation.
6. Commit generated artifacts to source control along with application code.
7. Use `eslint-plugin-relay`, especially the `relay/unused-fields` rule, for production readiness.

**Constraints and Limitations:**

- The compiler must run before the app can execute; it is not a runtime library.
- All GraphQL must be in `graphql` tagged templates; string concatenation breaks compilation.
- Generated artifacts must stay in sync with the schema; CI should fail on drift.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Compiler Setup and First Query

```graphql
# schema.graphql (excerpt)
type Query {
  viewer: Viewer
}

type Viewer {
  id: ID!
  name: String!
}
```

```tsx
// src/App.tsx
import { graphql, useLazyLoadQuery } from 'react-relay';

const AppQuery = graphql`
  query AppQuery {
    viewer {
      name
    }
  }
`;

function App() {
  const data = useLazyLoadQuery(AppQuery, {});
  return <h1>{data.viewer?.name}</h1>;
}
```

**Expected Output:** The compiler validates `AppQuery` against the schema, generates `AppQuery.graphql.ts` with the query text and types, and the runtime fetches and renders the viewer's name.

**Why this output:** The compiler parses the `graphql` tagged template, validates it, and emits the artifact. `useLazyLoadQuery` uses the generated artifact to fetch data at runtime.

#### Example 2: Watch Mode During Development

```bash
# package.json scripts
"relay": "relay-compiler --watch",
"dev": "concurrently \"npm run relay\" \"vite\""
```

**Expected Output:** The compiler watches source files and regenerates artifacts on save, logging `[relay]` output alongside the dev server.

**Why this output:** Watch mode recompiles on every GraphQL change, keeping artifacts and types in sync with the code.

### Real-World Cases

- **Large codebases:** Ahead-of-time validation catches GraphQL errors before runtime.
- **Type safety:** Generated types keep components in sync with the schema.
- **CI pipelines:** Compiler validation prevents broken queries from merging.

### References

- Relay — Installation and Setup — https://relay.dev/docs/getting-started/installation-and-setup/
- Relay — Relay Compiler — https://relay.dev/docs/guides/compiler/
- Relay — Production Setup — https://relay.dev/docs/guides/production-setup/

---

## 2. Relay Fragments, Queries & Mutations

### Definitions

**Core Definition:** Fragments declare a component's data requirements, queries fetch root data, and mutations modify server data with optional optimistic updates and store updaters.

**Technical Definition:** A **fragment** is a GraphQL fragment defined with `graphql` and read with `useFragment(fragment, fragmentRef)`, where `fragmentRef` is an opaque `$key` reference passed from a parent. A **query** is fetched with `usePreloadedQuery(query, preloadedQuery)` (preferred) or `useLazyLoadQuery(query, variables)`. A **mutation** is executed with `useMutation(mutation, config)`, which returns `[commitMutation, isInFlight]`; the config accepts `variables`, `optimisticResponse`, `optimisticUpdater`, `updater`, and `onCompleted`. Relay enforces globally unique operation names ending with the operation type (`Query`, `Mutation`, `Subscription`).

**Beginner-Friendly Explanation:** Fragments are the building blocks: each component says "I need these fields." Queries fetch the data for a whole page, pulling in all the fragments used by its child components. Mutations change data on the server and can update the cache immediately (optimistic) before the server responds.

### Purposes

- To colocate data requirements with the components that use them (fragments).
- To fetch all data for a page in a single network request (queries).
- To perform create/update/delete operations with optimistic UI (mutations).
- To mask data between components and prevent implicit coupling.
- To generate typed props and hooks automatically.

### Syntax Rules and Structure

**Fragment:**

```tsx
const UserFragment = graphql`
  fragment UserFragment_user on User {
    name
    email
  }
`;

function User({ user }: { user: UserFragment_user$key }) {
  const data = useFragment(UserFragment, user);
  return <h1>{data.name}</h1>;
}
```

**Query:**

```tsx
const AppQuery = graphql`
  query AppQuery($id: ID!) {
    user(id: $id) {
      ...UserFragment_user
    }
  }
`;

const data = usePreloadedQuery(AppQuery, preloadedQuery);
```

**Mutation:**

```tsx
const mutation = graphql`
  mutation CreateUserMutation($input: CreateUserInput!) {
    createUser(input: $input) {
      user {
        id
        name
      }
    }
  }
`;

const [commit, isInFlight] = useMutation(mutation);
commit({
  variables: { input: { name: 'Ada' } },
  optimisticUpdater: (store) => { /* ... */ },
  updater: (store) => { /* ... */ },
});
```

**Syntax Rules:**

1. Fragment names must be globally unique and end with the type they are on (convention: `ComponentName_propName`).
2. The fragment reference type is `<fragment_name>$key`; import it from the generated `.graphql.js` file.
3. Query root components should have a single query tightly coupled to the component.
4. Mutations should live in their own hook module named for what the mutation does (e.g., `useAddPostComment`).
5. `usePreloadedQuery` requires a `preloadedQuery` from `loadQuery` or `useQueryLoader`; `useLazyLoadQuery` fetches during render and is not preferred.
6. Use `@argumentDefinitions` on fragments and `@arguments` to pass values.

**Constraints and Limitations:**

- Data masking means parents cannot read child fragment fields directly.
- `useLazyLoadQuery` can cause waterfalls; prefer preloaded queries.
- Mutations do not automatically update the cache; use `updater` or declarative directives.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Fragment Composition

```tsx
// UserAvatar.tsx
const UserAvatarFragment = graphql`
  fragment UserAvatar_user on User {
    avatarUrl
    name
  }
`;

function UserAvatar({ user }: { user: UserAvatar_user$key }) {
  const data = useFragment(UserAvatarFragment, user);
  return <img src={data.avatarUrl} alt={data.name} />;
}
```

```tsx
// UserProfile.tsx
const UserProfileFragment = graphql`
  fragment UserProfile_user on User {
    name
    ...UserAvatar_user
  }
`;

function UserProfile({ user }: { user: UserProfile_user$key }) {
  const data = useFragment(UserProfileFragment, user);
  return (
    <div>
      <h1>{data.name}</h1>
      <UserAvatar user={data} />
    </div>
  );
}
```

**Expected Output:** `UserProfile` renders the name and the avatar. The compiler flattens both fragments into the parent query.

**Why this output:** Fragments compose. The parent fragment spreads the child fragment, and the compiler generates a query that requests all fields for one network request.

#### Example 2: Mutation with Updater

```tsx
const CreatePostMutation = graphql`
  mutation CreatePostMutation($input: CreatePostInput!) {
    createPost(input: $input) {
      postEdge {
        cursor
        node {
          id
          title
        }
      }
    }
  }
`;

function useCreatePost() {
  const [commit, isInFlight] = useMutation(CreatePostMutation);

  return (input: { title: string }) =>
    commit({
      variables: { input },
      updater: (store) => {
        const payload = store.getRootField('createPost');
        const newEdge = payload.getLinkedRecord('postEdge');
        const connection = ConnectionHandler.getConnection(
          store.getRoot(),
          'PostsList_posts'
        );
        if (connection && newEdge) {
          ConnectionHandler.insertEdgeAfter(connection, newEdge);
        }
      },
    });
}
```

**Expected Output:** Creating a post inserts the new edge into the cached connection, updating the list without a full refetch.

**Why this output:** The `updater` receives the store proxy and the server response, and imperatively inserts the new edge into the connection.

### Real-World Cases

- **Newsfeeds:** Fragment-based stories with nested comments and likes.
- **CRUD forms:** Mutations with optimistic updates and connection updates.
- **Dashboards:** Composed widgets each declaring their own fragments.

### References

- Relay — Organizing GraphQL Operations — https://relay.dev/docs/tutorial/organizing-mutations-queries-and-subscriptions/
- Relay — Thinking in Relay — https://relay.dev/docs/principles-and-architecture/thinking-in-relay/
- Relay — Fragments — https://relay.dev/docs/guided-tour/rendering/fragments/
- Relay — Queries — https://relay.dev/docs/guided-tour/rendering/queries/
- Relay — GraphQL Mutations — https://relay.dev/docs/guided-tour/updating-data/graphql-mutations/
- Relay — Relay Best Practices — https://github.com/facebook/relay/blob/main/skills/relay-best-practices/SKILL.md

---

## 3. Relay Pagination

### Definitions

**Core Definition:** Relay pagination uses the Connection abstraction — a standardized schema shape with `edges`, `node`, and `pageInfo` — together with the `@connection` and `@refetchable` directives and the `usePaginationFragment` hook to fetch pages of data incrementally.

**Technical Definition:** A Connection has three parts: `edges` (each with a `cursor` and a `node`), `node` (the actual item), and `pageInfo` (with `hasNextPage`, `endCursor`, etc.). `usePaginationFragment` requires the fragment to have a `@connection(key: "...")` directive on the connection field and a `@refetchable(queryName: "...")` directive to generate the pagination query automatically. The hook returns `data`, `loadNext`, `loadPrevious`, `hasNext`, `hasPrevious`, `isLoadingNext`, and `isLoadingPrevious`. `loadNext(count)` fetches the next page and Relay automatically appends the new items to the connection. For streaming pagination, `@stream_connection` can be used instead of `@connection`.

**Beginner-Friendly Explanation:** Pagination in Relay uses a standard format called a Connection. You mark your list field with `@connection` and `@refetchable`, and then `usePaginationFragment` gives you a `loadNext` function. Calling `loadNext(10)` fetches 10 more items and appends them to the list automatically.

### Purposes

- To paginate through large collections with cursor-based pagination.
- To load more items without refetching the entire list.
- To support forward, backward, and bidirectional pagination.
- To cache each page for instant back-navigation.
- To enable streaming pagination with `@stream_connection`.

### Syntax Rules and Structure

**Fragment with `@connection` and `@refetchable`:**

```tsx
const FriendsListFragment = graphql`
  fragment FriendsListComponent_user on User
    @argumentDefinitions(
      count: { type: "Int", defaultValue: 10 }
      cursor: { type: "String" }
    )
    @refetchable(queryName: "FriendsListPaginationQuery") {
    name
    friends(first: $count, after: $cursor)
      @connection(key: "FriendsList_user_friends") {
      edges {
        node {
          name
          age
        }
      }
    }
  }
`;

function FriendsList({ user }: { user: FriendsListComponent_user$key }) {
  const { data, loadNext, hasNext, isLoadingNext } = usePaginationFragment(
    FriendsListFragment,
    user
  );

  return (
    <>
      <h1>Friends of {data.name}</h1>
      <List items={data.friends?.edges.map((e) => e.node)} />
      {hasNext && (
        <button onClick={() => loadNext(10)} disabled={isLoadingNext}>
          Load more friends
        </button>
      )}
    </>
  );
}
```

**Syntax Rules:**

1. The fragment must have `@connection(key: "...")` on the connection field.
2. The fragment must have `@refetchable(queryName: "...")` to generate the pagination query.
3. `@refetchable` can only be added to fragments on `Viewer`, `Query`, or types implementing `Node` (types with an `id`).
4. The pagination variables (`first`, `after`) must be declared via `@argumentDefinitions`.
5. `loadNext(count)` fetches the next page; Relay appends items automatically.
6. `hasNext` and `isLoadingNext` control the "Load more" UI.
7. `loadPrevious` and `hasPrevious` support backward pagination.

**Constraints and Limitations:**

- Changing variables other than pagination variables during pagination does not make sense; it would query a different connection.
- `@refetchable` fragments must be on `Viewer`, `Query`, or `Node` types.
- The connection field must match the `@connection` key exactly.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Forward Pagination with `loadNext`

```tsx
const FeedFragment = graphql`
  fragment Feed_posts on Query
    @argumentDefinitions(
      count: { type: "Int", defaultValue: 10 }
      cursor: { type: "String" }
    )
    @refetchable(queryName: "FeedPaginationQuery") {
    posts(first: $count, after: $cursor)
      @connection(key: "Feed_posts") {
      edges {
        node {
          id
          title
        }
      }
    }
  }
`;

function Feed({ posts }: { posts: Feed_posts$key }) {
  const { data, loadNext, hasNext, isLoadingNext } =
    usePaginationFragment(FeedFragment, posts);

  return (
    <div>
      {data.posts?.edges.map(({ node }) => (
        <PostCard key={node.id} post={node} />
      ))}
      {hasNext && (
        <button onClick={() => loadNext(10)} disabled={isLoadingNext}>
          {isLoadingNext ? 'Loading…' : 'Load more'}
        </button>
      )}
    </div>
  );
}
```

**Expected Output:** The first 10 posts render; clicking "Load more" fetches the next 10 and appends them. The button disappears when `hasNext` is false.

**Why this output:** `loadNext(10)` fetches the next page using the `endCursor`. Relay automatically appends the new edges to the cached connection, and the component re-renders.

#### Example 2: Streaming Pagination with `@stream_connection`

```tsx
const StreamFeedFragment = graphql`
  fragment StreamFeed_posts on Query
    @argumentDefinitions(count: { type: "Int", defaultValue: 10 })
    @refetchable(queryName: "StreamFeedPaginationQuery") {
    posts(first: $count) @stream_connection(key: "StreamFeed_posts") {
      edges {
        node { id title }
      }
    }
  }
`;
```

**Expected Output:** The connection is fetched and each item is received incrementally as it becomes ready, rather than waiting for the entire page.

**Why this output:** `@stream_connection` accepts the same arguments as `@connection` plus streaming controls, enabling incremental data delivery for each item.

### Real-World Cases

- **Social feeds:** Infinite scroll with cursor pagination.
- **Comment threads:** Load more comments.
- **Product listings:** Paginated product grids.

### References

- Relay — Connections & Pagination — https://relay.dev/docs/tutorial/connections-pagination/
- Relay — usePaginationFragment — https://relay.dev/docs/api-reference/use-pagination-fragment/
- Relay — Pagination — https://relay.dev/docs/guided-tour/list-data/pagination/
- Relay — Streaming Pagination — https://relay.dev/docs/guided-tour/list-data/streaming-pagination/
- Relay — GraphQL Directives — https://github.com/facebook/relay/blob/main/website/versioned_docs/version-v13.0.0/api-reference/graphql/graphql-directives.md

---

## 4. Relay Store

### Definitions

**Core Definition:** The Relay store is a normalized, in-memory cache that keeps every GraphQL entity once, keyed by its ID, and notifies subscribed components when data changes.

**Technical Definition:** The store (`RelayModernStore`) uses a `RecordSource` — a flat, ID-keyed map of records — as its normalized cache. Each record has a `__typename`, an `id`, and fields. Relationships are stored as references to other records by ID. When data is written (from a query, mutation, or subscription), the store notifies all components subscribed to affected records, causing them to re-render. The store supports **garbage collection** to evict records no longer referenced by any view, **retention** via `Disposable` (records are retained until the disposable is disposed), and **imperative modification** through updater functions. Optimistic updates are applied to the store immediately and rolled back on mutation failure.

**Beginner-Friendly Explanation:** The Relay store is the smart cache that holds all your data. Every object is stored once, keyed by its ID. When data changes, every component using that object updates automatically. Old data is cleaned up automatically when nothing uses it anymore.

### Purposes

- To eliminate duplicate data and ensure consistency across the app.
- To automatically update every component that reads changed data.
- To support optimistic updates with automatic rollback.
- To enable complex client-side updates via updater functions.
- To manage memory with garbage collection and retention.
- To provide a single source of truth for all GraphQL data.

### Syntax Rules and Structure

**Store record shape:**

```js
{
  __typename: 'User',
  id: '123',
  name: 'Ada',
  friends: { __ref: 'User:456' },
}
```

**Updater types:**

| API | When to use |
|-----|-------------|
| `optimisticUpdater` | Apply changes immediately, rolled back on failure |
| `updater` | Apply changes after successful mutation |
| `commitLocalUpdate` | Imperatively modify local store data outside mutations |

**Garbage collection:** Relay runs GC to delete data no longer referenced by any component. Records retained via `Disposable` are exempt until disposed.

**Syntax Rules:**

1. All entities need a stable `id` and `__typename` for normalization.
2. Use `store.get(id)`, `store.getRootField(name)`, and `store.getLinkedRecord(field)` in updaters.
3. Use `ConnectionHandler.getConnection`, `insertEdgeAfter`, `deleteEdge`, and `ConnectionHandler` helpers for connection updates.
4. Optimistic updaters run when the mutation is triggered; regular updaters run on success.
5. `commitLocalUpdate` updaters do not receive a second argument and automatically notify subscribers.
6. If two optimistic responses modify the same value and the first is rolled back, the second remains applied; an optimistic updater would be re-run instead.

**Constraints and Limitations:**

- Direct store access is considered "unsafe" and should be used only when declarative directives cannot achieve the change.
- Multiple optimistic responses affecting the same value can produce unexpected results; optimistic updaters are safer for complex cases.
- Garbage collection can evict data that is not currently referenced; use retention if data must persist.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Optimistic Updater with Rollback

```tsx
const LikePostMutation = graphql`
  mutation LikePostMutation($input: LikePostInput!) {
    likePost(input: $input) {
      post {
        id
        likes
        liked
      }
    }
  }
`;

const [commit] = useMutation(LikePostMutation);

commit({
  variables: { input: { postId } },
  optimisticUpdater: (store) => {
    const post = store.get(postId);
    if (post) {
      post.setValue((post.getValue('likes') || 0) + 1, 'likes');
      post.setValue(true, 'liked');
    }
  },
  updater: (store) => {
    const payload = store.getRootField('likePost');
    const post = payload?.getLinkedRecord('post');
    if (post) {
      // The server response is written automatically; no manual write needed
    }
  },
});
```

**Expected Output:** The like count and liked state update immediately. If the mutation fails, the optimistic update is rolled back automatically. On success, the server response is written to the store.

**Why this output:** `optimisticUpdater` applies the change immediately. Relay rolls it back on error. `updater` runs on success and can perform additional updates.

#### Example 2: `commitLocalUpdate` for Local Data

```tsx
import { commitLocalUpdate } from 'react-relay';

function incrementCartCount(environment: Environment) {
  commitLocalUpdate(environment, (store) => {
    const cart = store.get('cart');
    if (cart) {
      cart.setValue((cart.getValue('count') || 0) + 1, 'count');
    }
  });
}
```

**Expected Output:** The cart count updates in the store and all subscribed components re-render immediately, without a network request.

**Why this output:** `commitLocalUpdate` imperatively modifies the store and automatically notifies components subscribed to the changed data.

### Real-World Cases

- **Likes and follows:** Optimistic updaters for instant feedback.
- **Cart counts:** Local updates without server round trips.
- **Complex list updates:** Manual connection manipulation after mutations.
- **Node invalidation:** Removing or resetting nodes with `store.invalidate`.

### References

- Relay — Imperatively modifying store data (unsafe) — https://relay.dev/docs/guided-tour/updating-data/imperatively-modifying-store-data-unsafe/
- Relay — Local Data Updates — https://relay.dev/docs/guided-tour/updating-data/local-data-updates/
- Relay — Runtime Architecture (garbage collection) — https://relay.dev/docs/principles-and-architecture/runtime-architecture/
- Relay — Relay Best Practices (normalized store) — https://github.com/facebook/relay/blob/main/skills/relay-best-practices/SKILL.md

---

## 5. Relay Best Practices

### Definitions

**Core Definition:** Relay best practices are the guidelines for structuring fragments, queries, mutations, and the store to build maintainable, performant, and type-safe Relay applications.

**Technical Definition:** Best practices centre on **co-location** (fragments live next to the component that uses them), **data masking discipline** (never bypass the fragment reference), **query placement at roots** (root components own a single query; hooks should not contain root queries), **preloaded queries over lazy queries** (`usePreloadedQuery` avoids waterfalls), **operation naming** (globally unique, suffixed with operation type), **mutation organisation** (mutations in their own hook modules named for their effect), **compiler in CI** (commit artifacts, run lint rules, validate schema), **connection key consistency**, **optimistic updater safety**, and **garbage collection awareness**.

**Beginner-Friendly Explanation:** Good Relay usage means putting each fragment next to its component, never cheating around data masking, using preloaded queries so data loads early, naming operations clearly, and keeping the compiler happy. Relay is opinionated, and following its conventions is what makes it powerful.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Colocate fragments** | Each component declares its own fragment next to its code |
| **Respect data masking** | Never read fields a component did not select |
| **Put queries at roots** | Root components own a single query; avoid root queries in hooks |
| **Prefer preloaded queries** | `loadQuery` + `usePreloadedQuery` over `useLazyLoadQuery` |
| **Use globally unique operation names** | End with `Query`, `Mutation`, or `Subscription` |
| **Organise mutations by effect** | `useAddPostComment` in its own module, not by component |
| **Commit generated artifacts** | Keep `.graphql.js` files in source control |
| **Run the compiler in CI** | Fail on schema drift and GraphQL errors |
| **Use `eslint-plugin-relay`** | Especially `relay/unused-fields` |
| **Keep connection keys consistent** | `@connection(key: "...")` must match usage |
| **Use optimistic updaters for complex updates** | Avoid multiple optimistic responses on the same value |
| **Be aware of garbage collection** | Use retention for data that must persist |
| **Use `@defer` and `@stream` for performance** | Incremental delivery for slow fields |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
// Co-located fragment
const UserCardFragment = graphql`
  fragment UserCard_user on User {
    id
    name
    ...UserAvatar_user
  }
`;

// Root query with preloaded query
const UserPageQuery = graphql`
  query UserPageQuery($id: ID!) {
    user(id: $id) {
      ...UserCard_user
    }
  }
`;

function UserPage({ queryRef }: { queryRef: PreloadedQuery<UserPageQuery> }) {
  const data = usePreloadedQuery(UserPageQuery, queryRef);
  return <UserCard user={data.user} />;
}

// Mutation in its own hook module
function useAddPostComment() {
  const [commit, isInFlight] = useMutation(AddPostCommentMutation);
  return { commit, isInFlight };
}
```

**❌ Bad Practices:**

```tsx
// ❌ Parent reading child fragment fields directly
function Parent({ user }) {
  return <h1>{user.name}</h1>; // Data masking: name is in UserCard_user
}

// ❌ Root query inside a hook
function useUser() {
  return useLazyLoadQuery(UserQuery, {}); // Waterfall risk
}

// ❌ Hand-written GraphQL types
interface User { id: string; name: string }; // Use generated types

// ❌ Mutation inside a component without its own module
function Component() {
  const [commit] = useMutation(Mutation); // Hard to reuse, poorly named
}
```

**Expected Output:** The good practices produce composable, type-safe, and performant Relay code. The bad practices violate data masking, introduce waterfalls, and bypass the compiler's benefits.

**Why this output:** Data masking prevents implicit coupling. Preloaded queries avoid waterfalls. Generated types stay in sync with the schema. Mutation modules are reusable and well-named.

### Real-World Cases

- **Large-scale applications:** Co-location and data masking keep components independent.
- **Performance-sensitive UIs:** Preloaded queries and `@defer` reduce waterfalls.
- **Team collaboration:** Consistent mutation organisation and naming reduce friction.

### References

- Relay — Relay Best Practices — https://github.com/facebook/relay/blob/main/skills/relay-best-practices/SKILL.md
- Relay — Organizing GraphQL Operations — https://relay.dev/docs/tutorial/organizing-mutations-queries-and-subscriptions/
- Relay — Production Setup — https://relay.dev/docs/guides/production-setup/
- Relay — Thinking in Relay — https://relay.dev/docs/principles-and-architecture/thinking-in-relay/
- Relay — Queries for Interactions (preloaded queries) — https://relay.dev/docs/guided-tour/rendering/queries/

---

## References

- Relay — Official Documentation — https://relay.dev/docs/
- Relay — Tutorial Intro — https://relay.dev/docs/v16.0.0/tutorial/intro/
- Relay — Installation and Setup — https://relay.dev/docs/getting-started/installation-and-setup/
- Relay — Relay Compiler — https://relay.dev/docs/guides/compiler/
- Relay — Production Setup — https://relay.dev/docs/guides/production-setup/
- Relay — Thinking in Relay — https://relay.dev/docs/principles-and-architecture/thinking-in-relay/
- Relay — Fragments — https://relay.dev/docs/guided-tour/rendering/fragments/
- Relay — Queries — https://relay.dev/docs/guided-tour/rendering/queries/
- Relay — GraphQL Mutations — https://relay.dev/docs/guided-tour/updating-data/graphql-mutations/
- Relay — Organizing GraphQL Operations — https://relay.dev/docs/tutorial/organizing-mutations-queries-and-subscriptions/
- Relay — Connections & Pagination — https://relay.dev/docs/v14.0.0/tutorial/connections-pagination/
- Relay — usePaginationFragment — https://relay.dev/docs/api-reference/use-pagination-fragment/
- Relay — Pagination — https://relay.dev/docs/v15.0.0/guided-tour/list-data/pagination/
- Relay — Streaming Pagination — https://relay.dev/docs/guided-tour/list-data/streaming-pagination/
- Relay — GraphQL Directives — https://github.com/facebook/relay/blob/main/website/versioned_docs/version-v13.0.0/api-reference/graphql/graphql-directives.md
- Relay — Imperatively modifying store data (unsafe) — https://relay.dev/docs/guided-tour/updating-data/imperatively-modifying-store-data-unsafe/
- Relay — Local Data Updates — https://relay.dev/docs/guided-tour/updating-data/local-data-updates/
- Relay — Runtime Architecture — https://relay.dev/docs/principles-and-architecture/runtime-architecture/
- Relay — Relay Best Practices — https://github.com/facebook/relay/blob/main/skills/relay-best-practices/SKILL.md
- Relay — Community Learning Resources — https://relay.dev/docs/community-learning-resources/
- Relay — Introducing Relay Hooks — https://relay.dev/blog/2021/03/09/introducing-relay-hooks/