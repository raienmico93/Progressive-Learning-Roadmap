# Apollo Client — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Apollo Client is a comprehensive state management library for JavaScript that enables you to manage both local and remote data with GraphQL, providing automatic normalized caching, declarative data fetching, optimistic UI updates, and real-time subscriptions.

**Technical Definition:** Apollo Client is a GraphQL client that sits between your React application and a GraphQL server. It manages the full data lifecycle: it sends queries, mutations, and subscriptions over a configurable **link chain** (composable middleware for authentication, retries, error handling, and transport selection); it normalizes responses into an **InMemoryCache** keyed by `__typename` and `keyFields`; it re-renders every component watching an affected cache entry when data changes; and it supports **optimistic responses** that apply before the server confirms, with automatic rollback on failure. React hooks (`useQuery`, `useMutation`, `useSubscription`, and Suspense variants) expose this machinery declaratively. Apollo Client 4.x moves React hooks to `@apollo/client/react`, introduces typed error classes (`CombinedGraphQLErrors`, `ServerError`), and adds `dataState` for type-safe query state.

**Beginner-Friendly Explanation:** Apollo Client is a tool that makes it easy to get data from a GraphQL server and keep it in sync with your React UI. You write a query, Apollo fetches it, stores the result in a smart cache, and updates your components automatically when the data changes. When you change data (a mutation), you can update the screen immediately (optimistic UI) and let the cache handle the rest. It also supports live updates (subscriptions) and local-only state. The biggest thing to understand is that Apollo's cache is the star: it stores every object once, keyed by its type and ID, and every component that reads that object re-renders when it changes.

### Key Characteristics

- **Normalized caching:** Every entity is stored once, keyed by `__typename` + `keyFields`; all queries referencing the same entity share one cache entry.
- **Declarative hooks:** `useQuery`, `useLazyQuery`, `useMutation`, `useSubscription`, `useSuspenseQuery`, `useFragment`, and more.
- **Link chain:** Composable middleware for auth, retries, error handling, batching, and transport (HTTP, WebSocket) selection.
- **Optimistic UI:** `optimisticResponse` applies changes instantly; the cache rolls back automatically on server error.
- **Pagination:** `fetchMore` and type policies (`relayStylePagination`, custom `merge` functions) handle cursor and offset pagination.
- **Local state:** Reactive variables (`makeVar`) and field policies integrate local-only fields into GraphQL queries.
- **Subscriptions:** WebSocket-based real-time updates via `split` links and `useSubscription`.
- **Type safety:** GraphQL Codegen generates operation types; hooks are fully typed with generics.
- **Suspense support:** `useSuspenseQuery`, `useLoadableQuery`, `useBackgroundQuery`, and `createQueryPreloader` (v3.9+).
- **v4 migration:** React hooks import from `@apollo/client/react`; `uri` on the client is removed (use `new HttpLink()`); typed error classes replace `ApolloError`.

### Prerequisites

- React function components, `useState`, and `useEffect`.
- GraphQL fundamentals: queries, mutations, subscriptions, schemas, fragments.
- JavaScript Promises and `async`/`await`.
- TypeScript (strongly recommended for generated types).
- A GraphQL server endpoint (or a mocked server for development).

### Related Programming Areas

- **GraphQL** — the query language and runtime.
- **State Management** — server state, local state, and cache policies.
- **Data Fetching** — request lifecycle, caching, and error handling.
- **Real-Time Systems** — subscriptions over WebSockets.
- **Performance** — normalized caching, batching, and cache policies.

---

## 1. Apollo Client

### Definitions

**Core Definition:** Apollo Client is the top-level client instance that holds the cache, the link chain, and the configuration for all GraphQL operations in the application.

**Technical Definition:** `new ApolloClient({ link, cache, ... })` creates the client. It requires a `link` (the network transport chain) and a `cache` (an `InMemoryCache` instance). It is provided to the React tree via `<ApolloProvider client={client}>`. The client exposes imperative methods (`query`, `mutate`, `subscribe`, `watchQuery`, `readQuery`, `writeQuery`, `refetchQueries`, `resetStore`) in addition to the hooks. In v4, the React hooks are imported from `@apollo/client/react`, and the `uri` option on the client is removed in favour of `new HttpLink({ uri })`.

**Beginner-Friendly Explanation:** The Apollo Client is the central object that knows where your GraphQL server is, how to talk to it, and how to store the results. You create it once and give it to your whole app.

### Purposes

- To configure the GraphQL endpoint, cache, and link chain.
- To provide a single source of truth for all GraphQL operations.
- To expose imperative cache and network methods outside React.
- To integrate with React via `ApolloProvider`.
- To support multiple environments (client-side, SSR, RSC) with per-request client instances.

### Syntax Rules and Structure

**General Syntax (v4):**

```tsx
import { ApolloClient, InMemoryCache, HttpLink } from '@apollo/client';
import { ApolloProvider } from '@apollo/client/react';

const client = new ApolloClient({
  link: new HttpLink({ uri: '/graphql' }),
  cache: new InMemoryCache(),
});

<ApolloProvider client={client}>
  <App />
</ApolloProvider>
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `link` | The network chain (e.g., `HttpLink`, composed links) |
| `cache` | The normalized cache (`InMemoryCache`) |
| `defaultOptions` | Default fetch/error policies for queries and mutations |
| `ssrMode` | Enables SSR-safe behaviour |
| `connectToDevTools` | Enables Apollo DevTools |

**Syntax Rules:**

1. Create the client at module level (or per request for SSR) — never inside React state.
2. Wrap the app with `ApolloProvider` at the root.
3. Use `new HttpLink({ uri })` in v4; the `uri` client option is removed.
4. Configure `defaultOptions` for consistent fetch and error policies.
5. For SSR, create a new client per request to avoid cross-request cache leaks.

**Constraints and Limitations:**

- A single client is shared across the app; per-request clients are required for SSR to prevent data leaks.
- The client is not a React state value; do not store it in `useState`.
- The v4 migration requires codemods and import path changes.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Client Setup with Auth and Error Links

```tsx
import {
  ApolloClient,
  InMemoryCache,
  HttpLink,
  from,
} from '@apollo/client';
import { onError } from '@apollo/client/link/error';
import { setContext } from '@apollo/client/link/context';

const httpLink = new HttpLink({ uri: '/graphql' });

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('token');
  return {
    headers: {
      ...headers,
      ...(token ? { authorization: `Bearer ${token}` } : {}),
    },
  };
});

const errorLink = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors) {
    graphQLErrors.forEach(({ message, path }) =>
      console.error(`[GraphQL error]: ${message} at ${path}`)
    );
  }
  if (networkError) {
    console.error(`[Network error]: ${networkError}`);
  }
});

export const client = new ApolloClient({
  link: from([errorLink, authLink, httpLink]),
  cache: new InMemoryCache(),
  defaultOptions: {
    watchQuery: { fetchPolicy: 'cache-and-network', errorPolicy: 'all' },
    query: { fetchPolicy: 'cache-first', errorPolicy: 'all' },
  },
});
```

**Expected Output:** Every request carries the bearer token; GraphQL and network errors are logged centrally.

**Why this output:** `from([...])` composes links in order. `setContext` adds the auth header. `onError` provides a central error hook. `defaultOptions` set the default policies.

#### Example 2: SSR-Safe Client Creation

```tsx
// lib/apollo-client.ts
import { ApolloClient, InMemoryCache, HttpLink } from '@apollo/client';

export function createApolloClient() {
  return new ApolloClient({
    ssrMode: true,
    link: new HttpLink({ uri: process.env.GRAPHQL_ENDPOINT }),
    cache: new InMemoryCache(),
  });
}
```

**Expected Output:** A new client is created per server request, preventing cache leakage between users.

**Why this output:** `ssrMode: true` disables certain browser-only behaviours. Creating a new client per request isolates the cache.

### Real-World Cases

- **SPAs:** A single client with auth and error links.
- **Next.js/SSR:** Per-request client creation with `ssrMode`.
- **Multi-endpoint apps:** `split` links to route operations to different servers.

### References

- Apollo Client 4.x Guide (apollographql/skills) — https://github.com/apollographql/skills/blob/HEAD/skills/apollo-client/SKILL.md#1
- Apollo Client 4.x Guide (clidey) — https://github.com/clidey/whodb/blob/1e5baef8c4e7e806ae3097847417fb320cf96144/.agents/skills/apollo-client/SKILL.md
- Apollo GraphQL Best Practices (Mindrally) — https://github.com/mindrally/skills/blob/HEAD/apollo-graphql/SKILL.md#1
- Apollo Client Patterns (agents-inc) — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-data-fetching-graphql-apollo/SKILL.md
- CoreUI — How to use React with GraphQL — https://coreui.io

---

## 2. GraphQL Queries, Mutations & Subscriptions

### Definitions

**Core Definition:** Queries read data, mutations change data, and subscriptions listen to real-time data streams — all defined in GraphQL and executed through Apollo Client hooks.

**Technical Definition:** `useQuery` subscribes a component to a GraphQL query, returning `loading`, `error`, `data`, `dataState` (v4), and a `refetch` function. `useLazyQuery` returns a tuple with an execute function for triggered queries. `useMutation` returns a tuple with a mutate function and status flags (`loading`, `error`, `data`). `useSubscription` subscribes to a WebSocket-backed GraphQL subscription. All hooks are fully typed with generics for variables and result data, and GraphQL Codegen generates those types automatically from the schema.

**Beginner-Friendly Explanation:** Queries get data, mutations change it, subscriptions watch it change in real time. Apollo gives you a hook for each.

### Purposes

- To fetch data declaratively with `useQuery`.
- To fetch on demand with `useLazyQuery`.
- To create, update, and delete data with `useMutation`.
- To receive real-time updates with `useSubscription`.
- To type all operations with generated TypeScript types.
- To handle loading, error, and empty states consistently.

### Syntax Rules and Structure

**General Syntax:**

```tsx
// Query
const { loading, error, data, dataState } = useQuery(GET_USER, { variables });

// Lazy query
const [execute, { loading, data }] = useLazyQuery(GET_USER);

// Mutation
const [mutate, { loading, error }] = useMutation(CREATE_USER);

// Subscription
const { data, loading } = useSubscription(USER_ADDED);
```

**Syntax Rules:**

1. Queries and mutations are defined with `gql` tagged templates.
2. Variables are passed via the `variables` option.
3. `useQuery` fetches automatically on mount; `useLazyQuery` waits for the execute function.
4. `useMutation` returns a tuple: the mutate function and the result object.
5. `useSubscription` requires a WebSocket link (typically via `split`).
6. In v4, import hooks from `@apollo/client/react`; `gql` comes from `@apollo/client`.
7. Use `dataState` (v4) for type-safe narrowing: `'empty' | 'loading' | 'partial' | 'complete' | 'error' | 'streaming'`.

**Constraints and Limitations:**

- `useSubscription` requires a WebSocket-capable server and a properly configured `split` link.
- `useMutation` does not automatically update the cache; you must use `update`, `refetchQueries`, or an optimistic response.
- Queries with missing variables will not execute; use `skip` to conditionally disable.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Typed Query with Codegen

```tsx
import { gql } from '@apollo/client';
import { useQuery } from '@apollo/client/react';
import type { GetUserQuery, GetUserQueryVariables } from '@/generated/graphql';

const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
    }
  }
`;

function UserProfile({ userId }: { userId: string }) {
  const { loading, error, data, dataState } = useQuery<
    GetUserQuery,
    GetUserQueryVariables
  >(GET_USER, { variables: { id: userId } });

  if (loading) return <ProfileSkeleton />;
  if (error) return <ErrorBanner error={error} />;
  if (dataState === 'complete' && data?.user) {
    return <ProfileCard user={data.user} />;
  }
  return <EmptyState />;
}
```

**Expected Output:** The profile renders with fully typed data. Loading and error states are handled explicitly.

**Why this output:** Generics type the variables and result. `dataState` provides type-safe narrowing in v4. The component renders each state.

#### Example 2: Mutation with Cache Update

```tsx
import { gql } from '@apollo/client';
import { useMutation } from '@apollo/client/react';
import type { CreateUserMutation, CreateUserMutationVariables } from '@/generated/graphql';

const CREATE_USER = gql`
  mutation CreateUser($input: CreateUserInput!) {
    createUser(input: $input) {
      id
      name
    }
  }
`;

function CreateUserForm() {
  const [createUser, { loading, error }] = useMutation<
    CreateUserMutation,
    CreateUserMutationVariables
  >(CREATE_USER, {
    refetchQueries: ['GetUsers'],
    awaitRefetchQueries: true,
  });

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    const name = new FormData(e.currentTarget).get('name') as string;
    await createUser({ variables: { input: { name } } });
  }

  return (
    <form onSubmit={handleSubmit}>
      {error && <p role="alert">{error.message}</p>}
      <input name="name" disabled={loading} />
      <button disabled={loading}>{loading ? 'Creating…' : 'Create'}</button>
    </form>
  );
}
```

**Expected Output:** The user is created and the `GetUsers` query refetches, updating the list.

**Why this output:** `useMutation` provides the mutate function and status flags. `refetchQueries` updates the list after the mutation completes.

#### Example 3: Subscription via WebSocket

```tsx
import { gql } from '@apollo/client';
import { useSubscription } from '@apollo/client/react';

const USER_ADDED = gql`
  subscription OnUserAdded {
    userAdded {
      id
      name
    }
  }
`;

function LiveUserFeed() {
  const { data, loading } = useSubscription(USER_ADDED);

  if (loading) return <Spinner />;
  return <FeedItem user={data?.userAdded} />;
}
```

**Expected Output:** New users appear in the feed as the server pushes them over WebSocket.

**Why this output:** `useSubscription` opens a persistent WebSocket connection and invokes the component on each event. The data is written to the normalized cache.

### Real-World Cases

- **Dashboards:** Queries for stats, mutations for filters, subscriptions for live data.
- **Chat apps:** Subscriptions for new messages, mutations for sending.
- **CRUD apps:** Queries for lists, mutations for create/update/delete.

### References

- Apollo Client 4.x Guide (apollographql/skills) — https://github.com/apollographql/skills/blob/HEAD/skills/apollo-client/SKILL.md#1
- Apollo Client Patterns (agents-inc) — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-data-fetching-graphql-apollo/SKILL.md
- Using Apollo with TypeScript (Apollo Docs) — https://www.apollographql.com/docs/react/v2/development-testing/static-typing
- CoreUI — How to use React with GraphQL — https://coreui.io

---

## 3. Apollo Cache & Local State

### Definitions

**Core Definition:** The Apollo cache is a normalized, in-memory store that keeps every GraphQL entity exactly once, keyed by its type and identifier; local state extends this cache with client-only fields and reactive variables.

**Technical Definition:** `InMemoryCache` normalizes responses by splitting them into individual objects identified by `__typename` + `keyFields` (default `id`). Queries that reference the same entity share one cache entry, and updating that entry updates every query that reads it. **Type policies** customise how a type is identified (`keyFields`), how a field is read/written (`read`/`merge`), and how many cache entries a paginated field gets (`keyArgs`). **Reactive variables** (`makeVar`) hold local-only state outside the cache; **field policies** can read from reactive variables to populate local-only fields, allowing GraphQL queries to mix remote and local data.

**Beginner-Friendly Explanation:** The Apollo cache is a smart storage that keeps each piece of data only once. If two components ask for the same user, they share the same cached user. Local state lets you store things that do not come from the server — like a theme toggle or a local filter — and still access them through the same GraphQL API.

### Purposes

- To eliminate duplicate data and ensure consistency across queries.
- To automatically update every component watching a changed entity.
- To support instant UI updates via optimistic responses and cache writes.
- To manage local-only state with reactive variables and field policies.
- To customise pagination and field merging with type policies.
- To improve performance by serving data directly from the cache.

### Syntax Rules and Structure

**Type policy configuration:**

```tsx
const cache = new InMemoryCache({
  typePolicies: {
    Product: {
      keyFields: ['sku'], // Custom identity instead of id
    },
    Query: {
      fields: {
        posts: {
          keyArgs: ['category'], // Separate cache entries per category
          merge(existing = [], incoming) {
            return [...existing, ...incoming];
          },
        },
      },
    },
  },
});
```

**Reactive variable:**

```tsx
import { makeVar, useReactiveVar } from '@apollo/client';

export const themeVar = makeVar<'light' | 'dark'>('light');

function ThemeToggle() {
  const theme = useReactiveVar(themeVar);
  return (
    <button onClick={() => themeVar(theme === 'light' ? 'dark' : 'light')}>
      {theme}
    </button>
  );
}
```

**Syntax Rules:**

1. Configure `typePolicies` in the `InMemoryCache` constructor.
2. `keyFields` must be defined for every entity type with a non-default identity.
3. `keyArgs` controls how many cache entries a paginated field gets.
4. `merge` functions combine existing and incoming data (essential for pagination).
5. `makeVar` creates a reactive variable; `useReactiveVar` subscribes React components to it.
6. Field policies can read from reactive variables to populate local-only fields.
7. Local resolvers are deprecated; use field policies and reactive variables instead.

**Constraints and Limitations:**

- Without `keyFields`, Apollo assumes `id` or `_id`; entities keyed by `sku`, `slug`, or composite keys normalize incorrectly.
- Optimistic responses missing `__typename` and the identifying field fail to normalize silently.
- Reactive variables are not stored in the cache and do not persist across reloads unless you add persistence.
- `merge` functions must be pure and handle `undefined` existing values.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Custom Key Fields and Merge for Pagination

```tsx
const cache = new InMemoryCache({
  typePolicies: {
    Product: {
      keyFields: ['sku'], // Products are identified by SKU
    },
    Query: {
      fields: {
        products: {
          keyArgs: ['category'], // Separate lists per category
          merge(existing = [], incoming) {
            return [...existing, ...incoming];
          },
        },
      },
    },
  },
});
```

**Expected Output:** Products are cached by SKU, and the `products` field accumulates pages per category instead of overwriting.

**Why this output:** `keyFields: ['sku']` ensures correct normalization. `keyArgs` separates cache entries per category. `merge` concatenates pages.

#### Example 2: Reactive Variable for Local State

```tsx
import { makeVar, useReactiveVar } from '@apollo/client';

export const cartCountVar = makeVar(0);

function CartBadge() {
  const count = useReactiveVar(cartCountVar);
  return <span>{count}</span>;
}

function AddToCartButton() {
  return (
    <button onClick={() => cartCountVar(cartCountVar() + 1)}>
      Add to cart
    </button>
  );
}
```

**Expected Output:** The cart badge updates instantly when the button is clicked, without a server request.

**Why this output:** `makeVar` creates a reactive variable; `useReactiveVar` subscribes the component. Changing the variable re-renders all subscribers.

### Real-World Cases

- **E-commerce:** Products keyed by SKU with category-specific pagination.
- **Theming:** Dark/light mode stored in a reactive variable.
- **Local filters:** Search and filter state kept outside the server cache.
- **Optimistic cart:** Cart count updated immediately via a reactive variable.

### References

- Managing local state (Apollo Docs) — https://www.apollographql.com/docs/react/v3/local-state/local-state-management
- Apollo Client Patterns (agents-inc) — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-data-fetching-graphql-apollo/SKILL.md
- Apollo Client 4.x Guide (apollographql/skills) — https://github.com/apollographql/skills/blob/HEAD/skills/apollo-client/SKILL.md#1
- Apollo Caching Strategies (LobeHub) — https://lobehub.com/pt-BR/skills/neversight-learn-skills.dev-web-data-fetching-graphql-apollo?activeTab=resources#1

---

## 4. Apollo Links

### Definitions

**Core Definition:** Apollo Links are composable middleware units that form a chain through which every GraphQL operation passes, enabling authentication, retries, error handling, batching, and transport selection.

**Technical Definition:** A link is an object that implements a `request` handler receiving `(operation, forward)` and returning an observable. `ApolloLink.from([...])` composes links in serial order. `ApolloLink.split(test, left, right)` routes operations conditionally based on a predicate. Common links include `HttpLink` (terminating HTTP transport), `setContext` (async header injection), `onError` (error interception), `RetryLink` (retry with backoff), `BatchHttpLink` (batching), and `GraphQLWsLink` (WebSocket subscriptions). The link chain always ends in a terminating link (usually `HttpLink` or a split between HTTP and WebSocket).

**Beginner-Friendly Explanation:** Apollo Links are like a pipeline that every GraphQL request flows through. You can add stages for authentication, retries, logging, and routing — and the request passes through each stage before reaching the server.

### Purposes

- To attach authentication headers to every request.
- To retry failed requests with backoff.
- To log and handle GraphQL and network errors centrally.
- To batch multiple operations into one request.
- To route queries and mutations over HTTP and subscriptions over WebSocket via `split`.
- To support multiple endpoints (e.g., different GraphQL servers for different domains).

### Syntax Rules and Structure

**Composing links:**

```tsx
import { ApolloLink, HttpLink, from } from '@apollo/client';
import { RetryLink } from '@apollo/client/link/retry';

const link = from([
  new RetryLink(),
  authLink,
  errorLink,
  new HttpLink({ uri: '/graphql' }),
]);
```

**Split for subscriptions:**

```tsx
import { split } from '@apollo/client';
import { getMainDefinition } from '@apollo/client/utilities';
import { GraphQLWsLink } from '@apollo/client/link/subscriptions';
import { createClient } from 'graphql-ws';

const wsLink = new GraphQLWsLink(createClient({ url: 'wss://api/graphql' }));

const splitLink = split(
  ({ query }) => {
    const def = getMainDefinition(query);
    return def.kind === 'OperationDefinition' && def.operation === 'subscription';
  },
  wsLink,
  httpLink
);
```

**Syntax Rules:**

1. `from([...])` executes links in order; the last must be terminating.
2. `split(test, left, right)` routes based on the test predicate.
3. `setContext` supports async header injection (e.g., token refresh).
4. `onError` receives `{ graphQLErrors, networkError, operation, forward }`.
5. `RetryLink` retries with exponential backoff by default.
6. The terminating link (`HttpLink`, `GraphQLWsLink`, or `BatchHttpLink`) must be last.

**Constraints and Limitations:**

- A link chain must have exactly one terminating link for each operation path.
- `split` requires a predicate that inspects the operation; complex routing logic should be extracted.
- Link order matters: retries before error logging means retried errors are logged multiple times.

### Annotated Code Examples

```tsx
import {
  ApolloClient,
  InMemoryCache,
  HttpLink,
  split,
} from '@apollo/client';
import { setContext } from '@apollo/client/link/context';
import { onError } from '@apollo/client/link/error';
import { RetryLink } from '@apollo/client/link/retry';
import { GraphQLWsLink } from '@apollo/client/link/subscriptions';
import { getMainDefinition } from '@apollo/client/utilities';
import { createClient } from 'graphql-ws';

const httpLink = new HttpLink({ uri: '/graphql' });

const wsLink = new GraphQLWsLink(
  createClient({ url: 'wss://api.example.com/graphql' })
);

const authLink = setContext((_, { headers }) => ({
  headers: {
    ...headers,
    authorization: `Bearer ${localStorage.getItem('token')}`,
  },
}));

const errorLink = onError(({ graphQLErrors, networkError }) => {
  graphQLErrors?.forEach((e) => console.error(e.message));
  if (networkError) console.error(networkError);
});

const retryLink = new RetryLink({ attempts: { max: 3 } });

const splitLink = split(
  ({ query }) => {
    const def = getMainDefinition(query);
    return (
      def.kind === 'OperationDefinition' && def.operation === 'subscription'
    );
  },
  wsLink,
  from([retryLink, errorLink, authLink, httpLink])
);

export const client = new ApolloClient({
  link: splitLink,
  cache: new InMemoryCache(),
});
```

**Expected Output:** Subscriptions go over WebSocket; queries and mutations go through retry, error logging, auth, and HTTP. Every HTTP request carries the bearer token.

**Why this output:** `split` routes by operation type. The HTTP branch composes retry, error, auth, and terminating links in order. The WebSocket branch handles subscriptions.

### Real-World Cases

- **JWT authentication:** `setContext` attaches tokens.
- **Real-time subscriptions:** `split` routes subscriptions to WebSocket.
- **Resilient requests:** `RetryLink` for transient failures.
- **Multi-endpoint apps:** `split` routes operations to different GraphQL servers.

### References

- ApolloLink API (Apollo Docs) — https://www.apollographql.com/docs/deploy-preview/6167777ba4697d8038d71f77/react/api/link/apollo-link
- Apollo Client 4.x Guide (apollographql/skills) — https://github.com/apollographql/skills/blob/HEAD/skills/apollo-client/SKILL.md#1
- Apollo GraphQL Best Practices (Mindrally) — https://github.com/mindrally/skills/blob/HEAD/apollo-graphql/SKILL.md#1

---

## 5. Apollo Optimistic UI

### Definitions

**Core Definition:** Apollo optimistic UI is the practice of applying a mutation's expected result to the cache immediately, before the server responds, with automatic rollback if the mutation fails.

**Technical Definition:** `useMutation` accepts an `optimisticResponse` option containing the expected server response. Apollo writes this response to the cache immediately and re-renders all components watching the affected data. When the real server response arrives, Apollo reconciles the cache with the actual result. If the mutation fails, the optimistic entry is discarded and the cache reverts to its previous state. The `update` callback can also manually modify the cache after the mutation. **Critical rule:** the `optimisticResponse` must include `__typename` and the identifying field (`id` or the configured `keyFields`) for normalization to succeed; without them, the optimistic write silently fails and the UI does not move.

**Beginner-Friendly Explanation:** Optimistic UI makes your app feel instant. When the user clicks "Like," you show the like immediately, assume the server will succeed, and undo it if something goes wrong.

### Purposes

- To provide instant feedback for mutations.
- To reduce perceived latency.
- To improve UX for likes, follows, toggles, and simple edits.
- To maintain correctness with automatic rollback on failure.
- To combine with `update` for cache updates that are not simple field changes.

### Syntax Rules and Structure

```tsx
const [addTodo] = useMutation(ADD_TODO, {
  optimisticResponse: {
    __typename: 'Mutation',
    addTodo: {
      __typename: 'Todo',
      id: 'temp-id',
      text: 'New todo',
      completed: false,
    },
  },
  update(cache, { data: { addTodo } }) {
    cache.modify({
      fields: {
        todos(existing = []) {
          const newTodoRef = cache.writeFragment({
            data: addTodo,
            fragment: gql`
              fragment NewTodo on Todo {
                id
                text
                completed
              }
            `,
          });
          return [...existing, newTodoRef];
        },
      },
    });
  },
});
```

**Syntax Rules:**

1. `optimisticResponse` must match the shape of the mutation result.
2. Include `__typename` on the mutation and every entity in the response.
3. Include the identifying field (`id` or `keyFields`) on every entity.
4. Use `update` to manually modify the cache after the mutation.
5. Use `refetchQueries` for a simple but network-costly refresh.
6. Apollo rolls back the optimistic write automatically on error.

**Constraints and Limitations:**

- Missing `__typename` or the identifying field causes silent normalization failure.
- Optimistic updates are a correctness trade-off; not suitable for critical mutations where failure is common.
- The optimistic response must be a plausible server response; complex server logic cannot be predicted.

### Annotated Code Examples

```tsx
const [likePost] = useMutation(LIKE_POST, {
  optimisticResponse: {
    __typename: 'Mutation',
    likePost: {
      __typename: 'Post',
      id: postId,
      liked: true,
      likes: currentLikes + 1,
    },
  },
  update(cache, { data }) {
    cache.modify({
      id: cache.identify({ __typename: 'Post', id: postId }),
      fields: {
        liked: () => data.likePost.liked,
        likes: () => data.likePost.likes,
      },
    });
  },
});
```

**Expected Output:** The like button updates instantly. If the mutation fails, the like reverts automatically. On success, the cache is reconciled with the server result.

**Why this output:** `optimisticResponse` applies the expected result immediately. `update` ensures the cache fields are correctly modified. Apollo rolls back on failure.

### Real-World Cases

- **Likes and reactions:** Instant toggling.
- **Todo lists:** Adding items before the server responds.
- **Follow buttons:** Immediate follower count updates.

### References

- web-data-fetching-graphql-apollo (LobeHub) — https://lobehub.com/pt-BR/skills/neversight-learn-skills.dev-web-data-fetching-graphql-apollo?activeTab=resources#1
- Apollo Client Patterns (agents-inc) — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-data-fetching-graphql-apollo/SKILL.md
- Apollo Caching Strategies (LobeHub) — https://lobehub.com/pt-BR/skills/neversight-learn-skills.dev-web-data-fetching-graphql-apollo?activeTab=resources#1
- Apollo Client 4.x Guide (apollographql/skills) — https://github.com/apollographql/skills/blob/HEAD/skills/apollo-client/SKILL.md#1

---

## 6. Apollo Pagination

### Definitions

**Core Definition:** Apollo pagination is the practice of fetching additional pages of data and merging them into the existing cached list using `fetchMore` and type policies.

**Technical Definition:** `fetchMore` accepts a `variables` object (the next cursor or offset) and an `updateQuery` function (or, in modern Apollo, a type policy `merge` function) that combines the previous result with the incoming page. For cursor-based pagination, `relayStylePagination` is a built-in helper that handles `edges`/`pageInfo` merging automatically. For offset-based pagination, a custom `merge` function in `keyArgs` and `merge` is required. Each paginated field must have a `keyArgs` policy so Apollo knows whether to create a new cache entry per filter or share one.

**Beginner-Friendly Explanation:** Pagination is how you load more items. Apollo's `fetchMore` fetches the next page and merges it into the list you already have, so the UI shows the accumulated items.

### Purposes

- To implement infinite scrolling and "Load more" pagination.
- To merge pages into a single cached list.
- To support cursor-based and offset-based pagination.
- To cache each page for instant back-navigation.
- To avoid refetching the entire list when loading more.

### Syntax Rules and Structure

**Cursor-based pagination with `fetchMore`:**

```tsx
const { data, fetchMore, loading } = useQuery(GET_POSTS, {
  variables: { cursor: null },
});

function loadMore() {
  fetchMore({
    variables: { cursor: data.posts.pageInfo.endCursor },
    updateQuery: (previous, { fetchMoreResult }) => {
      if (!fetchMoreResult) return previous;
      return {
        posts: {
          ...fetchMoreResult.posts,
          edges: [...previous.posts.edges, ...fetchMoreResult.posts.edges],
        },
      };
    },
  });
}
```

**Relay-style type policy:**

```tsx
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        posts: relayStylePagination(['category']),
      },
    },
  },
});
```

**Syntax Rules:**

1. Use `fetchMore` with the next cursor or offset in `variables`.
2. Use `updateQuery` (or a type policy `merge`) to combine pages.
3. For Relay-style connections, use `relayStylePagination` with `keyArgs`.
4. For offset pagination, use a custom `merge` function with `keyArgs` and `merge`.
5. Guard `fetchMore` calls with `hasNextPage` and `loading` to prevent duplicates.

**Constraints and Limitations:**

- Without a proper `merge` or `updateQuery`, `fetchMore` overwrites the list instead of appending.
- `relayStylePagination` only works with Relay-style connection shapes (`edges`, `pageInfo`).
- Offset pagination is unstable when items are inserted or deleted.

### Annotated Code Examples

```tsx
import { useQuery, gql } from '@apollo/client';

const GET_POSTS = gql`
  query GetPosts($cursor: String) {
    posts(first: 10, after: $cursor) {
      edges {
        node { id title }
      }
      pageInfo { endCursor hasNextPage }
    }
  }
`;

function PostList() {
  const { data, fetchMore, loading } = useQuery(GET_POSTS);

  async function loadMore() {
    if (!data?.posts.pageInfo.hasNextPage) return;
    await fetchMore({
      variables: { cursor: data.posts.pageInfo.endCursor },
      updateQuery: (prev, { fetchMoreResult }) => {
        if (!fetchMoreResult) return prev;
        return {
          posts: {
            ...fetchMoreResult.posts,
            edges: [...prev.posts.edges, ...fetchMoreResult.posts.edges],
          },
        };
      },
    });
  }

  return (
    <div>
      {data?.posts.edges.map(({ node }) => (
        <PostCard key={node.id} post={node} />
      ))}
      <button onClick={loadMore} disabled={loading}>
        {loading ? 'Loading…' : 'Load more'}
      </button>
    </div>
  );
}
```

**Expected Output:** The first page loads, and "Load more" fetches the next page and appends it to the existing list.

**Why this output:** `fetchMore` fetches the next page with the cursor. `updateQuery` merges the new edges into the previous list, so the cache and UI show the accumulated results.

### Real-World Cases

- **Social feeds:** Infinite scroll with cursor pagination.
- **Product listings:** Load more products.
- **Search results:** Accumulate results as the user scrolls.

### References

- web-data-fetching-graphql-apollo (LobeHub) — https://lobehub.com/pt-BR/skills/neversight-learn-skills.dev-web-data-fetching-graphql-apollo?activeTab=resources#1
- Apollo Client Patterns (agents-inc) — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-data-fetching-graphql-apollo/SKILL.md
- Apollo Caching Strategies (LobeHub) — https://lobehub.com/pt-BR/skills/neversight-learn-skills.dev-web-data-fetching-graphql-apollo?activeTab=resources#1
- Apollo Client 4.x Guide (apollographql/skills) — https://github.com/apollographql/skills/blob/HEAD/skills/apollo-client/SKILL.md#1

---

## 7. Apollo Best Practices

### Definitions

**Core Definition:** Apollo Client best practices are the guidelines for configuring the cache, structuring queries, handling mutations, and maintaining a predictable GraphQL data layer.

**Technical Definition:** Best practices combine **codegen discipline** (never hand-write GraphQL types), **cache identity** (`keyFields` on every entity, `keyArgs` on every paginated field), **normalization correctness** (`__typename` and identifying fields in every optimistic response), **error policy** (`errorPolicy: 'all'` for graceful degradation with partial data), **fetch policy** (`cache-first` for read-heavy data, `network-only` for real-time data), **mutation hygiene** (`update` or `refetchQueries` after mutations), **v4 migration** (hooks from `@apollo/client/react`, `new HttpLink()` instead of `uri`), and **client lifecycle** (never store the client in React state; create per request for SSR).

**Beginner-Friendly Explanation:** Good Apollo usage means generating your types, giving every entity a proper cache key, including `__typename` in optimistic responses, and choosing the right fetch policy for each query.

### Best Practices Table

| Practice | Description |
|----------|-------------|
| **Use GraphQL Codegen** | Never hand-write types; generate from the schema |
| **Configure `keyFields` for every entity** | Default `["id"]` is wrong for SKUs, slugs, and composite keys |
| **Set `keyArgs` for paginated fields** | Controls cache entry granularity |
| **Include `__typename` in optimistic responses** | Without it, normalization silently fails |
| **Include the identifying field in optimistic responses** | Required for cache normalization |
| **Use `errorPolicy: 'all'`** | Render partial data alongside errors |
| **Prefer `cache-first` for read-heavy data** | Serve from cache; refetch in background |
| **Use `network-only` for real-time data** | Always fetch fresh |
| **Update the cache after mutations** | Use `update`, `refetchQueries`, or optimistic response |
| **Never store Apollo Client in React state** | Use module-level or context |
| **Create a new client per SSR request** | Avoid cross-request cache leaks |
| **Use named constants for timeouts/retries** | No magic numbers |
| **Use Suspense hooks for route-level loading** | `useSuspenseQuery`, `useBackgroundQuery` |
| **Migrate to v4 imports** | Hooks from `@apollo/client/react`; `new HttpLink()` |

### Annotated Code Examples

**✅ Good Practices:**

```tsx
// Generated types
import type { GetUserQuery, GetUserQueryVariables } from '@/generated/graphql';

// KeyFields for non-default identity
const cache = new InMemoryCache({
  typePolicies: {
    Product: { keyFields: ['sku'] },
    Query: {
      fields: { products: relayStylePagination(['category']) },
    },
  },
});

// Optimistic response with __typename and id
useMutation(LIKE_POST, {
  optimisticResponse: {
    __typename: 'Mutation',
    likePost: { __typename: 'Post', id: postId, liked: true, likes: count + 1 },
  },
  update(cache, { data }) {
    cache.modify({
      id: cache.identify({ __typename: 'Post', id: postId }),
      fields: { liked: () => data.likePost.liked },
    });
  },
});

// Error policy for partial data
useQuery(GET_USER, { errorPolicy: 'all' });

// v4 imports
import { useQuery } from '@apollo/client/react';
```

**❌ Bad Practices:**

```tsx
// ❌ Hand-written types
interface UserData { user: { id: string; name: string } };

// ❌ Missing keyFields for SKU-keyed entities
const cache = new InMemoryCache(); // Products normalize by id, not sku

// ❌ Optimistic response missing __typename
optimisticResponse: { likePost: { id: postId, liked: true } };

// ❌ Storing client in React state
const [client] = useState(() => new ApolloClient(...));

// ❌ Using uri on the client in v4
new ApolloClient({ uri: '/graphql' }); // Use new HttpLink({ uri })
```

**Expected Output:** The good practices produce correct normalization, typed operations, and reliable optimistic updates. The bad practices cause silent cache misses, failed optimistic writes, and v4 migration breakage.

**Why this output:** Codegen keeps types in sync with the schema. `keyFields` ensures correct entity identity. `__typename` enables normalization. v4 requires the new import paths and link construction.

### Real-World Cases

- **All Apollo Client applications:** These practices apply universally.
- **E-commerce:** SKU-keyed products with category-specific pagination.
- **Real-time dashboards:** `network-only` fetch policy for live data.
- **SSR/Next.js:** Per-request client creation.

### References

- Apollo Client 4.x Guide (apollographql/skills) — https://github.com/apollographql/skills/blob/HEAD/skills/apollo-client/SKILL.md#1
- Apollo Client Patterns (agents-inc) — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-data-fetching-graphql-apollo/SKILL.md
- Apollo GraphQL Best Practices (Mindrally) — https://github.com/mindrally/skills/blob/HEAD/apollo-graphql/SKILL.md#1
- Apollo Client 4.x Guide (clidey) — https://github.com/clidey/whodb/blob/1e5baef8c4e7e806ae3097847417fb320cf96144/.agents/skills/apollo-client/SKILL.md
- CoreUI — How to use React with GraphQL — https://coreui.io
- Modern Apollo Client React (GraphQLConf 2026) — https://graphql.org

---

## References

- Apollo Client 4.x Guide (apollographql/skills) — https://github.com/apollographql/skills/blob/HEAD/skills/apollo-client/SKILL.md#1
- Apollo Client 4.x Guide (clidey) — https://github.com/clidey/whodb/blob/1e5baef8c4e7e806ae3097847417fb320cf96144/.agents/skills/apollo-client/SKILL.md
- Apollo Client Patterns (agents-inc) — https://github.com/agents-inc/skills/blob/845a1f2833eedc494bb82e3172710c06cab547a1/src/skills/web-data-fetching-graphql-apollo/SKILL.md
- Apollo GraphQL Best Practices (Mindrally) — https://github.com/mindrally/skills/blob/HEAD/apollo-graphql/SKILL.md#1
- web-data-fetching-graphql-apollo (LobeHub) — https://lobehub.com/pt-BR/skills/neversight-learn-skills.dev-web-data-fetching-graphql-apollo?activeTab=resources#1
- ApolloLink API (Apollo Docs) — https://www.apollographql.com/docs/deploy-preview/6167777ba4697d8038d71f77/react/api/link/apollo-link
- Managing local state (Apollo Docs) — https://www.apollographql.com/docs/react/v3/local-state/local-state-management
- Using Apollo with TypeScript (Apollo Docs) — https://www.apollographql.com/docs/react/v2/development-testing/static-typing
- CoreUI — How to use React with GraphQL — https://coreui.io
- Modern Apollo Client React (GraphQLConf 2026) — https://graphql.org
- Apollo Caching Strategies (LobeHub) — https://lobehub.com/pt-BR/skills/neversight-learn-skills.dev-web-data-fetching-graphql-apollo?activeTab=resources#1
- Apollo Client useQuery/Mutation/Subscription patterns (agents-inc) — https://github.com/agents-inc/skills/blob/main/src/skills/web-data-fetching-graphql-apollo/SKILL.md#1
- Apollo Client useQuery/Mutation/Subscription patterns (LobeHub) — https://lobehub.com/pt-BR/skills/neversight-learn-skills.dev-web-data-fetching-graphql-apollo?activeTab=resources#1