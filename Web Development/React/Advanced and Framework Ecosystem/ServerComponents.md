# React Server Components (RSC) Deep Dive: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** React Server Components (RSC) are a new type of React component that renders exclusively on the server, ships zero JavaScript to the browser, and can directly access server-only resources such as databases, file systems, and environment variables.

**Technical Definition:** React Server Components (RSC) is an application architecture that splits a component tree between a **server module graph** and a **client module graph**. Server Components render ahead of time — before bundling — in an environment separate from the client app or SSR server. They can run once at build time on a CI server, or for each request using a web server. Client Components, marked with the `'use client'` directive, run on both the server (for initial HTML) and the browser (for hydration and interactivity). The server renders the component tree into an **RSC Payload**, a serialized description of the UI that carries references to Client Components and the props passed to them. The client uses this payload to reconcile Server and Client Components into a single tree. RSC is stable in React 19 and is supported by frameworks like Next.js (App Router) and React Router v7 (Framework Mode, experimental).

**Beginner-Friendly Explanation:** Normally, all your React components run in the browser. With Server Components, some components run only on the server. They can talk directly to your database, use secret API keys, and don't send any JavaScript to the browser — making your app faster and more secure. But server components can't use `useState`, `onClick`, or any browser API. For interactive parts, you mark a file with `'use client'`, and that becomes the boundary: everything below it runs in the browser. The tricky part is that server components can only pass plain, serializable data to client components — no functions, no class instances.

### Key Characteristics

- **Module-Graph Boundary:** The server/client boundary is a **module boundary**, not a render boundary. When you add `'use client'` to a file, that file and everything it imports become part of the client bundle.
- **Zero Client JavaScript for Server Components:** Server Components never hydrate and never ship JavaScript to the browser. Their rendered output is HTML or a serialized RSC Payload.
- **Direct Backend Access:** Server Components can `await` data directly, import server-only modules (database clients, secret keys), and read environment variables without an API layer.
- **Serializable Props Only:** Data passed from Server to Client Components must be serializable by React's RSC serialization protocol, which is broader than JSON but excludes functions, class instances, and non-global symbols.
- **Children Composition Pattern:** A Client Component cannot import a Server Component, but a Server Component can pass another Server Component as a `children` prop to a Client Component. This is the "server-into-client" composition pattern.

### Prerequisites

- Solid understanding of React components, props, and JSX.
- Familiarity with JavaScript/TypeScript module systems and import/export syntax.
- Basic understanding of server-side rendering (SSR) and client-side hydration.
- Awareness of the difference between server-only and client-side code.
- Experience with a framework that supports RSC (Next.js App Router or React Router v7 Framework Mode).

### Related Programming Areas

- **Server-Side Rendering (SSR):** Rendering React to HTML on the server for initial page load.
- **Streaming SSR:** Sending HTML in progressive chunks over a single HTTP connection.
- **Module Graphs:** The dependency tree of modules in a JavaScript application.
- **Serialization Protocols:** The RSC Flight protocol for passing data across the server/client boundary.
- **Server Actions:** Functions marked with `'use server'` that can be called from Client Components.

### Core Concepts / Features

1. The RSC Architecture
2. The Serialization Bridge
3. Data-Access Patterns
4. Interleaving Server & Client

---

## Core Concept 1: The RSC Architecture

### Definitions

**Core Definition:** The RSC architecture is the fundamental separation of a React application into two module graphs — a server graph containing Server Components and a client graph containing Client Components — with a defined boundary between them.

**Technical Definition:** RSC splits a component tree between server and client module graphs. The server graph contains Server Components, which run exclusively on the server and ship no JavaScript. The client graph contains Client Components, which run on both the server (for initial HTML) and the browser (for hydration and interactivity). Next.js compiles a module used by both graphs separately for each environment. During rendering, the server graph produces references to Client Components and serializes the props passed to them. The client graph does not import the server graph; it receives references and serialized props through the RSC Payload. The `'use client'` directive creates a boundary in the module dependency tree, not in the render tree. A module can be part of both graphs if it is imported by both Server and Client Components (e.g., a pure utility function), but a module marked with `'use client'` and everything it imports becomes client-only.

**Beginner-Friendly Explanation:** Think of your app as two separate worlds: the server world and the browser world. Server Components live only in the server world — they can do anything the server can do, but they can't be interactive. Client Components live in both worlds — they render on the server to produce HTML, then "wake up" in the browser to handle clicks and typing. The `'use client'` directive is the door between the worlds. When you open that door in a file, everything inside that file (and everything it imports) crosses into the browser world.

### Purposes

- To keep server-only code (database queries, secrets, large dependencies) on the server, reducing JavaScript bundle size.
- To enable direct data access from components without building an API layer.
- To improve initial page load performance by shipping less JavaScript to the browser.
- To provide a clear, enforced boundary between server and client code.
- To enable streaming server rendering with progressive HTML delivery.
- To support React Server Components as the default rendering model in modern frameworks.

### Syntax Rules and Structure

**The `'use client'` Directive:**

```tsx
// app/hello.tsx — Server Component (default)
export default function Hello() {
  return <p>Hello from the server</p>;
}
```

```tsx
// app/counter.tsx — Client Component
'use client';
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>;
}
```

**Component Breakdown:**
- Server Components are the default. They run only on the server and ship no JavaScript.
- `'use client'` marks a file (and everything it imports) as a Client Component. The directive must be at the very top of the file, above any imports.
- Client Components run on both the server (for HTML) and the browser (for hydration and interactivity).
- Server Components cannot use `useState`, `useEffect`, `useContext`, or browser APIs.
- Client Components cannot directly access server-only resources like databases or secrets.

**The Server/Client Boundary:**

```
Server Component
├─ Server Component
└─ Client Component (boundary — 'use client')
    └─ Client Component
```

**Component Breakdown:**
- The boundary is at the module level, not the render tree level. When a Server Component imports a `'use client'` file, that import becomes the boundary.
- Everything below the boundary is part of the client bundle.
- A Client Component can render a Server Component only if the Server Component is passed as a `children` prop from a Server Component parent.

**Syntax Rules:**
- `'use client'` must be at the very top of the file, above any imports (comments are allowed above).
- `'use server'` marks a function as a Server Action, which is serializable and callable from Client Components.
- Server Components can import Client Components; Client Components cannot import Server Components directly (but can receive them as `children` props).
- Only serializable data can cross the Server → Client boundary.
- Define event handlers (`onClick`, `onChange`) inside Client Components, not in Server Components.
- Move `'use client'` as low in the component tree as possible to minimise the client bundle.
- Passing Server Components as `children` to Client Components allows Server Components to render inside Client Components without becoming client components.

**Constraints and Limitations:**
- Server Components cannot use React Hooks that depend on client state (`useState`, `useReducer`, `useEffect`, `useContext`).
- Server Components cannot use browser APIs (`window`, `document`, `localStorage`).
- Client Components cannot directly access server-only resources (databases, secrets).
- Non-serialisable props cause build-time or runtime errors at the boundary.
- The RSC model is framework-dependent; it works in Next.js App Router and other compatible frameworks, but not in plain React SPAs without a framework.

### Annotated Code Examples

**Example 1: Server Component Fetching Data and Passing to Client Component**

```tsx
// app/products/page.tsx — Server Component
import { db } from '@/lib/db';
import { ProductCard } from './ProductCard';

export default async function ProductsPage() {
  // Direct database access — no API layer needed
  const products = await db.product.findMany();

  return (
    <div>
      <h1>Products</h1>
      {products.map((product) => (
        <ProductCard
          key={product.id}
          id={product.id}
          name={product.name}
          price={product.price}
        />
      ))}
    </div>
  );
}
```

```tsx
// app/products/ProductCard.tsx — Client Component
'use client';
import { useState } from 'react';
import { addToCart } from '@/app/actions';

export function ProductCard({ id, name, price }: {
  id: string;
  name: string;
  price: number;
}) {
  const [adding, setAdding] = useState(false);

  async function handleAddToCart() {
    setAdding(true);
    await addToCart(id); // Server Action
    setAdding(false);
  }

  return (
    <div className="product-card">
      <h3>{name}</h3>
      <p>${price}</p>
      <button onClick={handleAddToCart} disabled={adding}>
        {adding ? 'Adding...' : 'Add to Cart'}
      </button>
    </div>
  );
}
```

```typescript
// app/actions.ts — Server Action
'use server';
import { db } from '@/lib/db';
import { revalidatePath } from 'next/cache';

export async function addToCart(productId: string) {
  await db.cartItem.create({ data: { productId } });
  revalidatePath('/cart');
}
```

**Expected Output:** The `ProductsPage` Server Component fetches products directly from the database and renders them. Each `ProductCard` is a Client Component that can be interacted with (the "Add to Cart" button). The `addToCart` Server Action is called from the client, executes on the server, and revalidates the cart path.

**Why This Output Occurs:** The `ProductsPage` runs only on the server and ships no JavaScript. The `ProductCard` is marked with `'use client'`, so it runs in the browser and can use `useState` and `onClick`. The boundary is crossed with serialisable props (`id`, `name`, `price` — all primitives). The `addToCart` Server Action is a serialisable function that the client can call over the network. This architecture keeps the database query server-side, ships minimal JavaScript, and enables interactivity where needed.

### Real-World Cases

- **E-commerce:** Server Components fetch product data directly from the database; Client Components handle "Add to Cart" interactions.
- **SaaS dashboards:** Server Components fetch analytics data; Client Components handle filters, charts, and interactive controls.
- **Content platforms:** Server Components render article content; Client Components handle comment forms and like buttons.
- **Enterprise applications:** Server Components access internal APIs and databases; Client Components handle user interactions and client-side state.

---

## Core Concept 2: The Serialization Bridge

### Definitions

**Core Definition:** The serialization bridge is the RSC protocol (the "Flight" protocol) that serializes data, props, and JSX elements passed from Server Components to Client Components, allowing data to cross the server/client boundary.

**Technical Definition:** The RSC serialization bridge is implemented by the React Flight protocol, which serializes the rendered output of Server Components into an **RSC Payload** — a JSON-like stream embedded in `<script>` tags alongside the server-rendered HTML. Props passed from Server Components to Client Components must be serializable by React's RSC serialization, which is **broader than JSON**. Serializable types include primitives (`string`, `number`, `bigint`, `boolean`, `undefined`, `null`), globally-registered symbols (`Symbol.for`), `Array`, `Map`, `Set`, `TypedArray`, `ArrayBuffer`, `Date`, plain objects, Promises, JSX elements, and Server Functions (`'use server'`). Non-serializable types include regular functions (not Server Actions), class instances, `WeakMap`, `WeakSet`, and non-global symbols (e.g., `Symbol('x')`). The serialized data directly impacts page weight and load time, so only pass fields that the client actually uses.

**Beginner-Friendly Explanation:** When a Server Component passes data to a Client Component, that data has to travel from the server to the browser. It can't send a function or a class instance — those don't survive the journey. So React serializes the data into a format that can be sent over the network and reconstructed on the other side. Think of it like packing a suitcase: you can pack clothes, books, and toiletries, but you can't pack a live animal. Only certain types of data are "packable." Dates, Maps, Sets, and plain objects are packable. Functions, class instances, and symbols are not.

### Purposes

- To enable data to cross the server/client boundary in a predictable, type-safe manner.
- To prevent runtime errors caused by attempting to pass non-serializable values.
- To keep the RSC Payload small by only serializing the data the client actually needs.
- To support JSX elements as serializable props, enabling the children composition pattern.
- To allow Server Actions (functions marked with `'use server'`) to be passed as props and called from the client.
- To provide a clear contract for what data can and cannot be shared across the boundary.

### Syntax Rules and Structure

**Serializable Types (Safe to Pass):**

| Type | Example |
|------|---------|
| Primitives | `string`, `number`, `bigint`, `boolean`, `undefined`, `null` |
| Global symbols | `Symbol.for('key')` |
| Collections | `Array`, `Map`, `Set` |
| Binary | `TypedArray`, `ArrayBuffer` |
| Date | `new Date()` |
| Plain objects | `{ id: 1, name: 'Game' }` |
| Promises | `Promise.resolve(data)` |
| JSX elements | `<ServerComponent />` |
| Server Functions | `'use server'` functions |

**Non-Serializable Types (Cause Errors):**

| Type | Example |
|------|---------|
| Regular functions | `() => {}` (not a Server Action) |
| Class instances | `new UserModel(data)` |
| Weak collections | `new WeakMap()`, `new WeakSet()` |
| Non-global symbols | `Symbol('x')` |

**General Syntax for Safe Serialization:**

```tsx
// ✅ SAFE: Serializable props
<ClientCard
  item={{ id: 1, name: 'Game' }}    // Plain object
  tags={['featured', 'sale']}        // Array of primitives
  isActive={true}                    // Boolean
  createdAt={new Date()}             // Date (serializable)
  metadata={new Map([['key', 'value']])} // Map (serializable)
  onAction={handleAction}             // Server Action ('use server')
/>

// ❌ ERROR: Non-serializable props
<ClientCard
  onClick={() => {}}                 // Regular function — cannot serialize
  formatter={new Intl.NumberFormat()} // Class instance — cannot serialize
  icon={Symbol('star')}              // Non-global symbol — cannot serialize
/>
```

**Component Breakdown:**
- Serializable types: primitives, plain objects/arrays, Server Actions, Date, Map, Set, TypedArray, ArrayBuffer.
- Non-serializable types: regular functions, class instances, non-global Symbols, WeakMap, WeakSet.
- Use Server Actions when you need to pass callable behaviour from Server to Client Components.
- Convert class instances to plain objects before passing: `{ url: myUrl.toString() }`.

**Syntax Rules:**
- Only serializable data can cross the Server → Client boundary.
- Functions must be Server Actions (`'use server'`) to be passed as props.
- Class instances must be converted to plain objects before passing.
- Use `Date`, `Map`, and `Set` directly — they are serializable and arrive as real instances on the client.
- Keep the RSC Payload small: only pass fields the client actually uses.
- Promises are serializable and can be passed as props (e.g., for `use()` on the client).
- JSX elements are serializable, enabling the children composition pattern.

**Constraints and Limitations:**
- Regular functions cause runtime errors or silent data loss at the boundary.
- Class instances lose their prototype and methods; only own enumerable properties survive.
- Non-global symbols cannot be serialized.
- Large RSC Payloads increase page weight and load time.
- The serialization protocol is an implementation detail of the RSC framework and may change between React versions.

### Annotated Code Examples

**Example 1: Serializing a Date and a Map Across the Boundary**

```tsx
// app/dashboard/page.tsx — Server Component
import { PostCard } from './PostCard';

export default async function Dashboard() {
  const post = await getPost();
  const metadata = new Map([
    ['views', post.views],
    ['likes', post.likes],
  ]);

  return (
    <PostCard
      title={post.title}
      createdAt={post.createdAt} // Date — serializable
      metadata={metadata}         // Map — serializable
    />
  );
}
```

```tsx
// app/dashboard/PostCard.tsx — Client Component
'use client';

export function PostCard({
  title,
  createdAt,
  metadata,
}: {
  title: string;
  createdAt: Date;
  metadata: Map<string, number>;
}) {
  // createdAt is a real Date object — can call .getFullYear()
  // metadata is a real Map — can call .get()
  return (
    <div>
      <h3>{title}</h3>
      <time>{createdAt.getFullYear()}</time>
      <p>Views: {metadata.get('views')}</p>
    </div>
  );
}
```

**Expected Output:** The `PostCard` receives a real `Date` object and a real `Map` object. It can call `createdAt.getFullYear()` and `metadata.get('views')` without conversion.

**Why This Output Occurs:** `Date` and `Map` are serializable by the RSC protocol. React serializes them on the server and reconstructs them as real instances on the client. No manual conversion (e.g., `toISOString()` or `Object.fromEntries()`) is required.

### Real-World Cases

- **E-commerce:** Passing product IDs, prices, and availability status to Client Components.
- **Dashboards:** Passing analytics data (timestamps, metrics) to Client Components for charting.
- **Content platforms:** Passing article metadata (dates, tags) to Client Components for display.
- **Multi-tenant apps:** Passing tenant configuration (plain objects) to Client Components.

---

## Core Concept 3: Data-Access Patterns

### Definitions

**Core Definition:** Data-access patterns in RSC are the techniques for fetching data directly from the server — through database queries, ORM integrations, and server-only modules — without building an API layer, while securing backend environment variables.

**Technical Definition:** Server Components run exclusively on the server and can `await` data directly in the component body. They can import server-only modules (database clients, ORMs, secret keys) that are never bundled into the client. The `server-only` package enforces this by throwing a build-time error if a module marked with `import 'server-only'` is imported into a Client Component. This enables a **direct data-access pattern**: instead of building API endpoints and fetching them from the client, Server Components query the database or call internal services directly, eliminating the client-to-server waterfall and reducing the attack surface. ORMs like Prisma, Drizzle, and Kysely integrate naturally with Server Components, providing type-safe queries that run on the server. Environment variables (API keys, database URLs) are accessed directly from `process.env` in Server Components and are never exposed to the browser.

**Beginner-Friendly Explanation:** Normally, to get data from a database, you'd build an API endpoint, then have your React component call that endpoint with `fetch`. With Server Components, you skip the API entirely. Your component can talk directly to the database — it's like having a direct phone line instead of going through a switchboard. This is faster, simpler, and more secure, because the database credentials never leave the server. The `server-only` package ensures that if you accidentally try to use server code in a Client Component, you get an error at build time instead of leaking secrets.

### Purposes

- To eliminate the API layer for server-rendered data, reducing complexity and latency.
- To enable type-safe database queries directly in components with ORMs.
- To prevent server-only code (database clients, secrets) from leaking into the client bundle.
- To secure backend environment variables by keeping them on the server.
- To reduce client-to-server waterfalls by fetching data during server render.
- To simplify authentication and authorisation checks by running them on the server.

### Syntax Rules and Structure

**Direct Database Access with Prisma:**

```tsx
// app/users/page.tsx — Server Component
import { db } from '@/lib/db'; // Prisma client

export default async function UsersPage() {
  // Direct database query — no API layer needed
  const users = await db.user.findMany({
    select: { id: true, name: true, email: true },
  });

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name} — {user.email}</li>
      ))}
    </ul>
  );
}
```

**Component Breakdown:**
- `db`: A Prisma client instance imported from `@/lib/db`.
- `db.user.findMany()`: A type-safe database query executed on the server.
- The result is serialized and passed to the component for rendering.
- No API endpoint, no `fetch`, no client-side loading state.

**Enforcing Server-Only Modules:**

```typescript
// lib/db.ts
import 'server-only'; // Throws if imported into a Client Component
import { PrismaClient } from '@prisma/client';

export const db = new PrismaClient();
```

**Component Breakdown:**
- `import 'server-only'`: A build-time guard that prevents the module from being imported into Client Components.
- If a Client Component attempts to import `db.ts`, the build fails with a clear error.
- This prevents accidental exposure of database credentials or server logic.

**Accessing Environment Variables:**

```tsx
// app/api/route.ts — Server Component or Route Handler
export async function GET() {
  const apiKey = process.env.SECRET_API_KEY; // Available on the server
  const data = await fetch(`https://api.example.com/data?key=${apiKey}`);
  return Response.json(await data.json());
}
```

**Component Breakdown:**
- `process.env.SECRET_API_KEY`: Accessed directly in server code.
- Environment variables are never sent to the browser.
- In Next.js, only variables prefixed with `NEXT_PUBLIC_` are exposed to the client.

**Syntax Rules:**
- Use `import 'server-only'` in modules that must never reach the client (database clients, secret utilities).
- Server Components can `await` data directly in the component body.
- ORMs (Prisma, Drizzle) integrate naturally with Server Components — import the client and query directly.
- Environment variables without the `NEXT_PUBLIC_` prefix are server-only.
- Use `cookies()` and `headers()` from `next/headers` to read request data in Server Components.
- Authentication and authorisation checks should run in Server Components or a Data Access Layer (DAL).

**Constraints and Limitations:**
- `server-only` throws at build time if imported into a Client Component.
- Server Components cannot use client-side state or effects for data fetching.
- Database connections must be managed carefully (connection pooling, serverless considerations).
- ORMs may not work in Edge Runtime if they rely on Node.js-specific APIs.
- The `server-only` package must be installed separately (`npm install server-only`).

### Annotated Code Examples

**Example 1: Complete Server-Only Data Access Module**

```typescript
// lib/db.ts
import 'server-only';
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient | undefined };

export const db =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query'] : [],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = db;
```

```tsx
// app/dashboard/page.tsx — Server Component
import { db } from '@/lib/db';
import { DashboardClient } from './DashboardClient';

export default async function DashboardPage() {
  // Direct database queries — run on the server
  const [userCount, recentOrders] = await Promise.all([
    db.user.count(),
    db.order.findMany({ take: 10, orderBy: { createdAt: 'desc' } }),
  ]);

  return (
    <DashboardClient
      userCount={userCount}
      recentOrders={recentOrders.map((o) => ({
        id: o.id,
        total: o.total,
        createdAt: o.createdAt.toISOString(),
      }))}
    />
  );
}
```

```tsx
// app/dashboard/DashboardClient.tsx — Client Component
'use client';

export function DashboardClient({
  userCount,
  recentOrders,
}: {
  userCount: number;
  recentOrders: { id: string; total: number; createdAt: string }[];
}) {
  return (
    <div>
      <p>Total users: {userCount}</p>
      <ul>
        {recentOrders.map((order) => (
          <li key={order.id}>Order {order.id} — ${order.total}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Expected Output:** The dashboard page fetches the user count and recent orders directly from the database. The data is passed to the Client Component, which renders it. The database credentials and Prisma client are never included in the client bundle.

**Why This Output Occurs:** The `db.ts` module is marked with `import 'server-only'`, so it cannot be imported into a Client Component. The `DashboardPage` Server Component queries the database directly and passes plain serializable data (numbers, strings) to the `DashboardClient` Client Component. The Prisma client stays on the server.

### Real-World Cases

- **E-commerce:** Server Components query product catalogs, inventory, and pricing from the database.
- **SaaS dashboards:** Server Components fetch user metrics, analytics, and usage data directly.
- **Content platforms:** Server Components read articles, comments, and metadata from a CMS or database.
- **Financial applications:** Server Components access transaction data with strict server-only enforcement.
- **Multi-tenant apps:** Server Components query tenant-specific data with tenant ID from cookies or headers.

---

## Core Concept 4: Interleaving Server & Client

### Definitions

**Core Definition:** Interleaving Server and Client Components is the pattern of nesting Server Components inside Client Components (or vice versa) using the `children` composition pattern, allowing interactive UI to wrap server-rendered content.

**Technical Definition:** A Client Component cannot import a Server Component directly, because doing so would require a new request back to the server. Instead, the **children composition pattern** allows a Server Component to pass another Server Component (or a Server-rendered element) as a `children` prop to a Client Component. From the Client Component's perspective, the `children` prop is already-rendered output — it receives a serialized RSC reference, not the Server Component's code. When a request is made, all Server Components are rendered first, including those nested inside Client Components. The rendered result (RSC Payload) contains references to the locations of Client Components. On the client, React uses the RSC Payload to reconcile Server and Client Components into a single tree. This pattern is essential for building layouts and wrappers that need interactivity (e.g., a modal, accordion, or tab component) while containing server-rendered content.

**Beginner-Friendly Explanation:** You can't put a Server Component inside a Client Component by importing it — that would break the boundary. But you can pass a Server Component as a child. Think of it like this: a Client Component is a box with a lid that can open and close (interactivity). You can't build a Server Component inside the box, but you can put an already-built Server Component into the box from the outside. The Client Component doesn't need to know how the Server Component was built — it just renders what it's given.

### Purposes

- To enable Client Components to wrap Server Components without importing them.
- To build interactive layouts (modals, tabs, accordions) that contain server-rendered content.
- To avoid the limitation that Client Components cannot import Server Components.
- To preserve the server/client boundary while composing complex UIs.
- To allow Server Components to be rendered inside Client Component wrappers without becoming client components.
- To enable patterns like "client-side interactive shell with server-rendered content slots."

### Syntax Rules and Structure

**General Syntax for the Children Composition Pattern:**

```tsx
// app/page.tsx — Server Component
import { ClientWrapper } from './ClientWrapper';
import { ServerContent } from './ServerContent';

export default function Page() {
  return (
    <ClientWrapper>
      <ServerContent /> {/* Server Component passed as children */}
    </ClientWrapper>
  );
}
```

```tsx
// app/ClientWrapper.tsx — Client Component
'use client';
import { useState } from 'react';

export function ClientWrapper({ children }: { children: React.ReactNode }) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? 'Hide' : 'Show'}
      </button>
      {isOpen && <div>{children}</div>}
    </div>
  );
}
```

**Component Breakdown:**
- `ClientWrapper` is a Client Component that manages `isOpen` state.
- `ServerContent` is a Server Component that renders server-only data.
- The `Page` Server Component passes `<ServerContent />` as `children` to `ClientWrapper`.
- From `ClientWrapper`'s perspective, `children` is already-rendered output (a serialized RSC reference).

**General Syntax for Named Slots:**

```tsx
// app/page.tsx — Server Component
import { Tabs } from './Tabs';
import { TabContent } from './TabContent';

export default function Page() {
  return (
    <Tabs
      tabs={[
        { label: 'Overview', content: <TabContent section="overview" /> },
        { label: 'Settings', content: <TabContent section="settings" /> },
      ]}
    />
  );
}
```

```tsx
// app/Tabs.tsx — Client Component
'use client';
import { useState } from 'react';

export function Tabs({ tabs }: { tabs: { label: string; content: React.ReactNode }[] }) {
  const [active, setActive] = useState(0);

  return (
    <div>
      <div role="tablist">
        {tabs.map((tab, i) => (
          <button key={i} role="tab" onClick={() => setActive(i)}>
            {tab.label}
          </button>
        ))}
      </div>
      <div role="tabpanel">{tabs[active].content}</div>
    </div>
  );
}
```

**Component Breakdown:**
- `Tabs` is a Client Component that manages the active tab state.
- Each tab's `content` is a Server Component (`<TabContent />`) passed as a prop.
- The Server Components are rendered on the server and passed as serialized JSX elements.

**Syntax Rules:**
- A Client Component cannot import a Server Component directly.
- A Server Component can pass a Server Component as a `children` prop or any other prop to a Client Component.
- The Client Component receives the Server Component's rendered output as a serialized RSC reference.
- Use `children` for a single content slot; use named props for multiple slots.
- JSX elements are serializable, so passing `<ServerComponent />` as a prop works.
- When a new request is made, all Server Components are rendered first, including those nested inside Client Components.

**Constraints and Limitations:**
- The Client Component cannot access the Server Component's props or state; it only receives the rendered output.
- The Server Component passed as children is rendered on the server, so it cannot use client-side state.
- Named slots increase the size of the RSC Payload if the content is large.
- The children composition pattern is the only way to nest Server Components inside Client Components.
- Client Components can still render Client Components anywhere in the tree.

### Annotated Code Examples

**Example 1: Modal with Server-Rendered Content**

```tsx
// app/products/page.tsx — Server Component
import { Modal } from './Modal';
import { ProductDetails } from './ProductDetails';

export default async function ProductsPage() {
  const product = await getProduct();

  return (
    <Modal>
      <ProductDetails product={product} />
    </Modal>
  );
}
```

```tsx
// app/products/Modal.tsx — Client Component
'use client';
import { useState } from 'react';

export function Modal({ children }: { children: React.ReactNode }) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>View Details</button>
      {isOpen && (
        <div className="modal-overlay">
          <div className="modal-content">
            <button onClick={() => setIsOpen(false)}>Close</button>
            {children}
          </div>
        </div>
      )}
    </div>
  );
}
```

```tsx
// app/products/ProductDetails.tsx — Server Component
export async function ProductDetails({ product }: { product: Product }) {
  const reviews = await getReviews(product.id);

  return (
    <div>
      <h2>{product.name}</h2>
      <p>{product.description}</p>
      <h3>Reviews ({reviews.length})</h3>
      <ul>{reviews.map((r) => <li key={r.id}>{r.text}</li>)}</ul>
    </div>
  );
}
```

**Expected Output:** The page renders a "View Details" button. Clicking it opens a modal containing the product details and reviews, which were rendered on the server. The modal itself is interactive (open/close), but its content is server-rendered.

**Why This Output Occurs:** The `Modal` Client Component manages the open/close state. The `ProductDetails` Server Component is passed as `children` to `Modal`. From `Modal`'s perspective, `children` is already-rendered output. The Server Component fetches reviews on the server, and the modal displays them without shipping the review-fetching code to the client.

**Example 2: Tabs with Server-Rendered Panels**

```tsx
// app/dashboard/page.tsx — Server Component
import { Tabs } from './Tabs';
import { AnalyticsPanel } from './AnalyticsPanel';
import { SettingsPanel } from './SettingsPanel';

export default function Dashboard() {
  return (
    <Tabs
      tabs={[
        { label: 'Analytics', content: <AnalyticsPanel /> },
        { label: 'Settings', content: <SettingsPanel /> },
      ]}
    />
  );
}
```

```tsx
// app/dashboard/Tabs.tsx — Client Component
'use client';
import { useState } from 'react';

export function Tabs({ tabs }: { tabs: { label: string; content: React.ReactNode }[] }) {
  const [active, setActive] = useState(0);

  return (
    <div>
      <div role="tablist">
        {tabs.map((tab, i) => (
          <button
            key={i}
            role="tab"
            aria-selected={active === i}
            onClick={() => setActive(i)}
          >
            {tab.label}
          </button>
        ))}
      </div>
      <div role="tabpanel">{tabs[active].content}</div>
    </div>
  );
}
```

**Expected Output:** The dashboard renders a tabbed interface. Clicking "Analytics" shows the analytics panel (server-rendered), and clicking "Settings" shows the settings panel (server-rendered). The tab switching is interactive, but the panel content is rendered on the server.

**Why This Output Occurs:** The `Tabs` Client Component manages the active tab state. Each panel (`AnalyticsPanel`, `SettingsPanel`) is a Server Component passed as `content` in the tabs array. The Server Components are rendered on the server and passed as serialized JSX elements. The Client Component switches between them without needing to import or know about the Server Components.

### Real-World Cases

- **E-commerce:** Interactive modals for product quick-view with server-rendered product details.
- **SaaS dashboards:** Tabbed interfaces where each tab's content is a Server Component fetching its own data.
- **Content platforms:** Accordion components that expand to show server-rendered article sections.
- **Social media:** Modal dialogs for post details with server-rendered comments.
- **Multi-step forms:** Client-side step navigation with server-rendered step content.

---

## References

- Guides: Server and Client Boundary – Next.js: https://nextjs.org/docs/app/guides/server-and-client-boundary
- React Server Components – React: https://az.react.dev/reference/rsc/server-components
- RSC Boundaries – GitHub (laguagu/claude-code-nextjs-skills): https://raw.githubusercontent.com/laguagu/claude-code-nextjs-skills/0f7357fcd11bcbad4ec9d1101b2acd4353b9865e/skills/next-best-practices/rsc-boundaries.md
- Composing Server and Client Components: The Modern React's Superpower – Epic React: https://www.epicreact.dev/composing-server-and-client-components-the-modern-reacts-superpower-08yn9
- React Server Components and Server Actions | React with TypeScript – Steve Kinney: https://stevekinney.com/courses/react-typescript/server-components-and-server-actions
- Server Components and Streaming SSR – Steve Kinney: https://stevekinney.com/courses/enterprise-ui/server-components-and-streaming-ssr
- React Server Components – React Router (Experimental): https://mintlify.wiki/react-router/react-server-components
- React Server Components | React Performance – Steve Kinney: https://stevekinney.com/courses/react-performance/react-server-components
- How to Optimize RSC Payload Size – Vercel: https://vercel.com/kb/how-to-optimize-rsc-payload-size
- RSC Serialization – GitHub (yonatangross/orchestkit): https://raw.githubusercontent.com/yonatangross/orchestkit/dc3bfe80565e68fb4674b5e2873d4ce96105e2da/plugins/ork/skills/react-server-components-framework/rules/rsc-serialization.md
- Interleaving Server and Client Components – Stack Overflow: https://stackoverflow.com/questions/78984381/interleaving-server-and-client-components
- Who Owns the Tree? RSC as a Protocol, Not an Architecture – TanStack Blog: https://tanstack.com/blog/who-owns-the-tree-rsc-as-a-protocol-not-an-architecture
- Server Components – React (mintlify.wiki): https://mintlify.wiki/facebook/react/reference/rsc/server-components
- React Server Components (Experimental) – React Router: https://mintlify.wiki/react-router/react-server-components