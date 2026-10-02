# Type-Safe Application Architecture: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Type-safe application architecture is the discipline of designing an entire React application—its shared types, domain models, state, forms, API contracts, validation schemas, and third-party integrations—so that TypeScript enforces correctness across every layer and boundary.

**Technical Definition:** Type-safe application architecture applies TypeScript's type system as a cross-cutting concern that unifies data models, state management, network contracts, form schemas, and library integrations. It encompasses: (1) **shared types and global declarations** — structured `.d.ts` files, module augmentation, and explicit imports that prevent type leakage; (2) **domain models and state management** — typed Redux Toolkit slices, Zustand stores, and context values that mirror the domain; (3) **form types and dynamic inputs** — generic multi-step form typing, React Hook Form integration, and discriminated unions for dynamic fields; (4) **API contracts and route parameters** — typed URL params, query strings, request/response envelopes, and standard error shapes; (5) **validation schemas** — runtime validators (Zod, Yup, ArkType) that produce TypeScript types via inference (`z.infer`, `InferType`, `infer`); and (6) **ecosystem integration** — handling third-party library types, declaration merging, and overriding missing or incorrect types. The goal is a single source of truth for each piece of data, propagated through the entire application without duplication or drift.

**Beginner-Friendly Explanation:** In a large app, the same data—a user, an order, a form—travels through many places: the API, the store, the components, the forms, the URL. If each place defines its own version of that data, they drift apart and cause bugs. Type-safe application architecture means defining each piece of data once and letting TypeScript carry it everywhere. If the backend changes a field name, your form, your store, and your component all break at compile time—before anyone ships a bug.

### Key Characteristics

- **Single Source of Truth:** Each data shape is defined once and derived everywhere else (via `Pick`, `Omit`, `z.infer`, `ReturnType`).
- **Boundary Discipline:** Types flow from the boundary (API, URL, form) inward; runtime validation happens at the boundary, and the rest of the app trusts the validated type.
- **Explicit Over Ambient:** Prefer explicit imports over global declarations; ambient `.d.ts` files are reserved for library augmentation and truly global types.
- **Domain-Driven Types:** Types mirror business concepts (`User`, `Order`, `Cart`), not database tables or API responses.
- **Schema-First Validation:** Zod, Yup, and ArkType schemas are the source of truth; TypeScript types are inferred from them.
- **Integration Hygiene:** Third-party library types are wrapped, augmented, or overridden at a single module, not scattered across the codebase.
- **Layer Independence:** Each layer (data, state, UI) depends on shared types, not on each other's internals.

### Prerequisites

- Solid understanding of React function components, JSX, and Hooks.
- Working knowledge of TypeScript fundamentals (types, interfaces, generics, unions, narrowing).
- Familiarity with advanced React TypeScript patterns (generic components, discriminated unions, utility types).
- Working knowledge of a state management library (Redux Toolkit, Zustand, or Context).
- Familiarity with a form library (React Hook Form) and a validation library (Zod, Yup).
- Basic understanding of routing (React Router) and API clients (fetch, Axios, TanStack Query).

### Related Programming Areas

- **Domain-Driven Design:** Bounded contexts and domain models.
- **API Design:** REST, GraphQL, OpenAPI, and tRPC contracts.
- **State Management:** Redux Toolkit, Zustand, Jotai, and Context.
- **Form Architecture:** React Hook Form, Formik, and schema validation.
- **Runtime Validation:** Zod, Yup, ArkType, Valibot, and io-ts.

### Core Concepts / Features

1. Shared Types & Global Declarations
2. Domain Models & State Management
3. Form Types & Dynamic Inputs
4. API Contracts & Route Parameters
5. Validation Schemas
6. Ecosystem Integration

---

## Core Concept 1: Shared Types & Global Declarations

### Definitions

**Core Definition:** Shared types are TypeScript types used across multiple modules and layers of an application; global declarations are ambient `.d.ts` files that augment the global scope or third-party modules.

**Technical Definition:** Shared types live in a `types/` or `domain/` directory and are imported explicitly via relative or path-aliased imports (`import type { User } from '@/domain/user'`). They are the single source of truth for domain models, API envelopes, and common utility types. Global declarations (`.d.ts` files) are ambient modules that do not export anything; they augment the global scope (`declare global {}`), extend third-party modules (`declare module 'library' {}`), or declare non-TypeScript assets (`declare module '*.svg'`). The key distinction: **explicit imports are preferred** for application types because they make dependencies visible and enable tree-shaking of unused types; **global declarations are reserved** for library augmentation, asset modules, and truly global runtime globals (`window.__APP_VERSION__`). TypeScript's `type` imports (`import type`) are erased at compile time and should be used when importing types only. Path aliases (`@/`) in `tsconfig.json` make shared imports consistent and refactor-safe.

**Beginner-Friendly Explanation:** Shared types are types everyone uses—like a `User` type. They live in one place and are imported by components, stores, and forms. Global declarations are different: they change TypeScript's behaviour globally—like telling it that `*.svg` files are modules, or adding a `__APP_VERSION__` property to `window`. For application types, always import them explicitly so you can see where they come from. For third-party library augmentation, use global declarations.

### Purposes

- To define domain types once and reuse them across layers.
- To make dependencies explicit through imports.
- To augment third-party libraries with missing or corrected types.
- To declare non-TypeScript assets as modules.
- To add application-specific properties to global objects (`window`, `globalThis`).
- To avoid type drift by centralising shared types.

### Syntax Rules and Structure

**Project Structure:**
```
src/
├── domain/              # Shared domain types
│   ├── user.ts
│   ├── order.ts
│   └── index.ts         # Barrel export
├── types/               # Cross-cutting types
│   ├── api.ts           # ApiResponse<T>, ApiError
│   ├── brand.ts         # Branded types
│   └── index.ts
├── global.d.ts          # Global declarations
├── env.d.ts             # Environment variables
└── assets.d.ts          # Asset module declarations
```

**Explicit Shared Types (Preferred):**
```typescript
// src/domain/user.ts
export type User = {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: string;
};

export type CreateUserInput = Pick<User, 'name' | 'email'>;
export type UserRole = User['role'];
```

```typescript
// src/domain/index.ts — barrel export
export type { User, CreateUserInput, UserRole } from './user';
export type { Order, OrderItem, OrderStatus } from './order';
```

```typescript
// Usage in a component
import type { User } from '@/domain';
```

**`import type` (Erased at Compile Time):**
```typescript
// ✅ Types are erased; no runtime import
import type { User } from '@/domain';

// ❌ Imports a runtime value (may fail if the module has no runtime export)
import { User } from '@/domain';
```

**Global Declaration (`global.d.ts`):**
```typescript
// src/global.d.ts
export {}; // Ensure this is a module

declare global {
  interface Window {
    __APP_VERSION__: string;
    __ANALYTICS__?: {
      track: (event: string, data?: Record<string, unknown>) => void;
    };
  }

  // Augment the NodeJS namespace
  namespace NodeJS {
    interface ProcessEnv {
      NODE_ENV: 'development' | 'production' | 'test';
      NEXT_PUBLIC_API_URL: string;
    }
  }
}
```

**Module Augmentation:**
```typescript
// src/types/react-router.d.ts
import 'react-router';

declare module 'react-router' {
  interface FutureConfig {
    v7_partialHydration?: boolean;
  }
}
```

**Asset Module Declarations:**
```typescript
// src/assets.d.ts
declare module '*.svg' {
  import type { FC, SVGProps } from 'react';
  const ReactComponent: FC<SVGProps<SVGSVGElement>>;
  export default ReactComponent;
}

declare module '*.module.css' {
  const classes: { readonly [key: string]: string };
  export default classes;
}

declare module '*.png' {
  const src: string;
  export default src;
}
```

**Environment Variables:**
```typescript
// src/env.d.ts
declare namespace NodeJS {
  interface ProcessEnv {
    readonly NODE_ENV: 'development' | 'production' | 'test';
    readonly NEXT_PUBLIC_API_URL: string;
    readonly NEXT_PUBLIC_SENTRY_DSN?: string;
  }
}
```

**Syntax Rules:**
- Store shared domain types in a dedicated `domain/` directory with a barrel export.
- Use `import type` when importing only types.
- Use path aliases (`@/domain`) configured in `tsconfig.json` for consistent imports.
- Use `.d.ts` files for global declarations, module augmentation, and asset modules.
- Add `export {}` at the top of a `.d.ts` file that uses `declare global` to make it a module.
- Never declare application types in `.d.ts` files; use explicit exports instead.
- Augment third-party libraries in a dedicated `types/library-name.d.ts` file.
- Declare environment variables in a single `env.d.ts` file.

**Constraints and Limitations:**
- Global declarations apply to the entire project; a mistake affects every file.
- `declare global` requires the file to be a module (`export {}`).
- Module augmentation must match the module's exact structure; incorrect augmentation silently fails.
- Path aliases require `tsconfig.json` and bundler configuration (Vite `resolve.alias`, Webpack `resolve.alias`).
- `.d.ts` files are not compiled; errors in them may not surface until another file uses the declarations.
- Barrel exports can hurt tree-shaking if not configured properly; use `import type` for type-only barrels.

### Annotated Code Examples

**Example 1: Shared Domain Types with Barrel Export**

```typescript
// src/domain/user.ts
export type UserRole = 'admin' | 'user' | 'guest';

export type User = {
  id: number;
  name: string;
  email: string;
  role: UserRole;
  createdAt: string;
};

export type CreateUserInput = Pick<User, 'name' | 'email' | 'role'>;
export type UpdateUserInput = Partial<Omit<User, 'id' | 'createdAt'>>;
```

```typescript
// src/domain/index.ts
export type { User, UserRole, CreateUserInput, UpdateUserInput } from './user';
export type { Order, OrderItem, OrderStatus } from './order';
export type { ApiResponse, ApiError } from './api';
```

```typescript
// src/features/users/UserProfile.tsx
import type { User } from '@/domain';

export function UserProfile({ user }: { user: User }) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <span>{user.role}</span>
    </div>
  );
}
```

**Expected Output:** The `UserProfile` component receives a `User` from the shared domain module. TypeScript ensures that `user.name`, `user.email`, and `user.role` are all valid properties, and that `user.role` is one of the three allowed literals.

**Why This Output Occurs:** The `User` type is defined once in `domain/user.ts`. The barrel export (`domain/index.ts`) re-exports it. The component imports the type explicitly with `import type`. Any change to the `User` type propagates to every consumer.

**Example 2: Global Augmentation for `window` and Environment Variables**

```typescript
// src/global.d.ts
export {};

declare global {
  interface Window {
    __APP_VERSION__: string;
    __ANALYTICS__?: {
      track: (event: string, data?: Record<string, unknown>) => void;
    };
  }
}

declare namespace NodeJS {
  interface ProcessEnv {
    readonly NODE_ENV: 'development' | 'production' | 'test';
    readonly NEXT_PUBLIC_API_URL: string;
  }
}
```

```typescript
// src/main.tsx
window.__APP_VERSION__ = '1.2.3';
window.__ANALYTICS__?.track('app_started', { version: window.__APP_VERSION__ });

const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

**Expected Output:** `window.__APP_VERSION__` and `window.__ANALYTICS__` are typed; `process.env.NEXT_PUBLIC_API_URL` is a string; `process.env.NODE_ENV` is one of three literals. TypeScript errors if you access `window.__UNKNOWN__` or assign a non-string to `__APP_VERSION__`.

**Why This Output Occurs:** The `declare global` block augments the `Window` interface, and the `declare namespace NodeJS` block augments `ProcessEnv`. All consumers of `window` and `process.env` see the augmented types.

### Real-World Cases

- **Design systems:** Shared `Theme`, `ColorToken`, and `SpacingToken` types.
- **Domain models:** `User`, `Order`, `Product`, `Cart` shared across features.
- **API envelopes:** `ApiResponse<T>`, `ApiError`, `Paginated<T>` shared across all API calls.
- **Environment variables:** Typed `process.env` for build-time configuration.
- **Asset modules:** Typed imports for SVGs, CSS Modules, and images.
- **Library augmentation:** Adding missing types to a third-party library.

### References

- TypeScript Handbook – Modules - https://www.typescriptlang.org/docs/handbook/2/modules.html
- TypeScript Handbook – Declaration Files - https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html
- TypeScript Handbook – Global `.d.ts` - https://www.typescriptlang.org/docs/handbook/declaration-files/templates/global-d-ts.html
- TypeScript Handbook – Module Augmentation - https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation
- TypeScript Handbook – `import type` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html#type-only-imports-and-export
- TypeScript Handbook – Path Mapping - https://www.typescriptlang.org/tsconfig#paths

---

## Core Concept 2: Domain Models & State Management

### Definitions

**Core Definition:** Domain models are TypeScript types that represent business concepts; state management is the store (Redux Toolkit, Zustand, Context) that holds and updates those models with full type safety.

**Technical Definition:** Domain models describe the shape of business entities (`User`, `Order`, `Cart`) and their operations. State management libraries are typed to enforce that state updates match the domain model. Redux Toolkit uses `createSlice` with typed `PayloadAction<T>` and `createAsyncThunk<Returned, ThunkArg>`; selectors use `RootState` for typed access. Zustand uses `create<State>()(...)` with a typed store and selector functions `useStore((s) => s.slice)`. Context uses `createContext<T | null>(null)` with a throwing custom hook. In all cases, the domain model is the single source of truth; the store holds instances of it, and components read typed slices. Async operations (thunks, sagas, TanStack Query) return typed data that is stored in the slice. Normalisation (via `createEntityAdapter`) types entities and IDs. The key architectural decision is **where** state lives: server state (TanStack Query), client state (Redux/Zustand), URL state (`useSearchParams`), and form state (React Hook Form) each have their own tooling, and types must flow between them.

**Beginner-Friendly Explanation:** Your app's state is a collection of business objects—users, orders, carts. Type-safe state management means the store knows exactly what shape each object has, and every action, selector, and update is checked by TypeScript. In Redux, this means typing actions and selectors. In Zustand, it means typing the store and its selectors. The benefit is that when you rename a field on the `User` type, every place that reads or writes that field breaks at compile time.

### Purposes

- To type the store's state as domain models.
- To type actions and reducers in Redux Toolkit.
- To type selectors for correct state access.
- To type async thunks and their return values.
- To type Zustand stores and their selectors.
- To type Context values with strict providers.
- To integrate server state (TanStack Query) with client state.

### Syntax Rules and Structure

**Redux Toolkit Slice:**
```typescript
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import type { User } from '@/domain';

type UsersState = {
  entities: User[];
  selectedId: number | null;
  status: 'idle' | 'loading' | 'success' | 'error';
  error: string | null;
};

const initialState: UsersState = {
  entities: [],
  selectedId: null,
  status: 'idle',
  error: null,
};

const usersSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {
    userAdded(state, action: PayloadAction<User>) {
      state.entities.push(action.payload);
    },
    userSelected(state, action: PayloadAction<number>) {
      state.selectedId = action.payload;
    },
    userUpdated(state, action: PayloadAction<User>) {
      const index = state.entities.findIndex((u) => u.id === action.payload.id);
      if (index !== -1) state.entities[index] = action.payload;
    },
  },
});

export const { userAdded, userSelected, userUpdated } = usersSlice.actions;
export default usersSlice.reducer;
```

**Component Breakdown:**
- `UsersState`: The slice's state shape, using the `User` domain model.
- `PayloadAction<User>`: Types the action's payload.
- Reducers mutate the draft state (Immer handles immutability).

**Typed Store and Selectors:**
```typescript
import { configureStore } from '@reduxjs/toolkit';
import usersReducer from './usersSlice';

export const store = configureStore({
  reducer: { users: usersReducer },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

```typescript
import { useSelector, useDispatch } from 'react-redux';
import type { RootState, AppDispatch } from './store';

// Typed hooks
export const useAppSelector = useSelector.withTypes<RootState>();
export const useAppDispatch = useDispatch.withTypes<AppDispatch>();

// Usage
function UserList() {
  const users = useAppSelector((state) => state.users.entities);
  // users: User[]
  return <ul>{users.map((u) => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

**Component Breakdown:**
- `RootState`: The full store shape, derived from the store.
- `AppDispatch`: The dispatch type.
- `withTypes<T>()`: The modern way to type `useSelector` and `useDispatch`.

**Typed Async Thunk:**
```typescript
import { createAsyncThunk } from '@reduxjs/toolkit';
import type { User, CreateUserInput } from '@/domain';

export const fetchUser = createAsyncThunk<User, number>(
  'users/fetchUser',
  async (userId, { rejectWithValue }) => {
    const res = await fetch(`/api/users/${userId}`);
    if (!res.ok) return rejectWithValue(`HTTP ${res.status}`);
    return (await res.json()) as User;
  }
);

export const createUser = createAsyncThunk<User, CreateUserInput>(
  'users/createUser',
  async (input) => {
    const res = await fetch('/api/users', {
      method: 'POST',
      body: JSON.stringify(input),
      headers: { 'Content-Type': 'application/json' },
    });
    return (await res.json()) as User;
  }
);
```

**Component Breakdown:**
- `createAsyncThunk<User, number>`: The return type is `User`, the argument type is `number`.
- `rejectWithValue`: Types the rejection value.

**Zustand Store with Domain Model:**
```typescript
import { create } from 'zustand';
import type { User } from '@/domain';

type UserStore = {
  users: User[];
  selectedId: number | null;
  addUser: (user: User) => void;
  selectUser: (id: number) => void;
  updateUser: (user: User) => void;
};

export const useUserStore = create<UserStore>((set) => ({
  users: [],
  selectedId: null,
  addUser: (user) => set((s) => ({ users: [...s.users, user] })),
  selectUser: (id) => set({ selectedId: id }),
  updateUser: (user) =>
    set((s) => ({ users: s.users.map((u) => (u.id === user.id ? user : u)) })),
}));

// Usage with selectors
function UserList() {
  const users = useUserStore((s) => s.users);
  return <ul>{users.map((u) => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

**Component Breakdown:**
- `UserStore`: The store's shape, using the `User` domain model.
- `create<UserStore>((set) => ...)`: Types the store creator.
- Selectors preserve the type of each slice.

**Context with Domain Model:**
```tsx
import { createContext, useContext } from 'react';
import type { User } from '@/domain';

type AuthContextValue = {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
};

const AuthContext = createContext<AuthContextValue | null>(null);

export function useAuth(): AuthContextValue {
  const context = useContext(AuthContext);
  if (context === null) throw new Error('useAuth must be used within AuthProvider');
  return context;
}
```

**Component Breakdown:**
- `AuthContextValue`: The context's shape, using the `User` domain model.
- `useAuth`: A throwing custom hook that returns a non-null value.

**Syntax Rules:**
- Derive the store type from the domain model; do not redefine entities in the store.
- Use `PayloadAction<T>` to type reducer payloads.
- Use `createAsyncThunk<Returned, ThunkArg>` to type async operations.
- Derive `RootState` and `AppDispatch` from the store; do not annotate manually.
- Use `useSelector.withTypes<RootState>()` and `useDispatch.withTypes<AppDispatch>()` for typed hooks.
- In Zustand, type the store with `create<State>()(...)`.
- In Context, use `createContext<T | null>(null)` with a throwing hook.
- Separate server state (TanStack Query) from client state (Redux/Zustand); do not duplicate.

**Constraints and Limitations:**
- Redux Toolkit requires `immer` for draft mutations; incorrect mutations cause runtime errors.
- `createAsyncThunk` cannot be typed for multiple return shapes; use discriminated unions in the payload.
- Zustand selectors must return stable references; use `useShallow` for object/array selectors.
- Context values must be memoised to prevent consumer re-renders.
- Store shape changes are breaking changes; version types carefully.
- Server state should not be stored in Redux/Zustand; use TanStack Query for caching and synchronisation.

### Annotated Code Examples

**Example 1: Redux Toolkit Slice with Domain Models**

```typescript
// src/features/users/usersSlice.ts
import { createSlice, createAsyncThunk, type PayloadAction } from '@reduxjs/toolkit';
import type { User, CreateUserInput } from '@/domain';

type UsersState = {
  entities: User[];
  selectedId: number | null;
  status: 'idle' | 'loading' | 'success' | 'error';
  error: string | null;
};

const initialState: UsersState = {
  entities: [],
  selectedId: null,
  status: 'idle',
  error: null,
};

export const fetchUsers = createAsyncThunk<User[]>('users/fetchUsers', async () => {
  const res = await fetch('/api/users');
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return (await res.json()) as User[];
});

export const createUser = createAsyncThunk<User, CreateUserInput>(
  'users/createUser',
  async (input) => {
    const res = await fetch('/api/users', {
      method: 'POST',
      body: JSON.stringify(input),
      headers: { 'Content-Type': 'application/json' },
    });
    return (await res.json()) as User;
  }
);

const usersSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {
    userSelected(state, action: PayloadAction<number>) {
      state.selectedId = action.payload;
    },
    userRemoved(state, action: PayloadAction<number>) {
      state.entities = state.entities.filter((u) => u.id !== action.payload);
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUsers.fulfilled, (state, action: PayloadAction<User[]>) => {
        state.status = 'success';
        state.entities = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.status = 'error';
        state.error = action.error.message ?? 'Failed to fetch users';
      })
      .addCase(createUser.fulfilled, (state, action: PayloadAction<User>) => {
        state.entities.push(action.payload);
      });
  },
});

export const { userSelected, userRemoved } = usersSlice.actions;
export default usersSlice.reducer;
```

**Expected Output:** The slice holds `User[]`, and all actions, reducers, and async thunks are typed. `fetchUsers.fulfilled` types the payload as `User[]`; `createUser.fulfilled` types it as `User`. Components can select `state.users.entities` with full type safety.

**Why This Output Occurs:** The `UsersState` type uses the shared `User` domain model. `PayloadAction<User[]>` and `PayloadAction<User>` type the reducers. `createAsyncThunk<User[], void>` and `createAsyncThunk<User, CreateUserInput>` type the async operations. `extraReducers` narrows the action types.

**Example 2: Zustand Store with Domain Models and Shallow Selectors**

```typescript
// src/features/cart/cartStore.ts
import { create } from 'zustand';
import { useShallow } from 'zustand/react/shallow';
import type { CartItem, Product } from '@/domain';

type CartState = {
  items: CartItem[];
  addItem: (product: Product) => void;
  removeItem: (productId: number) => void;
  clear: () => void;
  total: () => number;
};

export const useCartStore = create<CartState>((set, get) => ({
  items: [],
  addItem: (product) =>
    set((s) => {
      const existing = s.items.find((i) => i.productId === product.id);
      if (existing) {
        return {
          items: s.items.map((i) =>
            i.productId === product.id ? { ...i, quantity: i.quantity + 1 } : i
          ),
        };
      }
      return { items: [...s.items, { productId: product.id, quantity: 1, price: product.price }] };
    }),
  removeItem: (productId) =>
    set((s) => ({ items: s.items.filter((i) => i.productId !== productId) })),
  clear: () => set({ items: [] }),
  total: () => get().items.reduce((sum, i) => sum + i.price * i.quantity, 0),
}));

// Component with shallow selector
function CartSummary() {
  const { items, removeItem } = useCartStore(
    useShallow((s) => ({ items: s.items, removeItem: s.removeItem }))
  );
  const total = useCartStore((s) => s.total());
  return (
    <div>
      <ul>
        {items.map((item) => (
          <li key={item.productId}>
            {item.productId} × {item.quantity}
            <button onClick={() => removeItem(item.productId)}>Remove</button>
          </li>
        ))}
      </ul>
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}
```

**Expected Output:** A cart summary that shows items, quantities, and total. TypeScript types `items` as `CartItem[]`, `removeItem` as `(productId: number) => void`, and `total` as `number`. The `useShallow` selector prevents re-renders when unrelated state changes.

**Why This Output Occurs:** The `CartState` type uses `CartItem[]` from the domain. `create<CartState>((set, get) => ...)` types the store. Selectors preserve types, and `useShallow` compares the selected object shallowly.

### Real-World Cases

- **E-commerce:** Cart, products, orders, and user state.
- **SaaS dashboards:** Analytics, billing, settings, and notification state.
- **Multi-tenant apps:** Tenant, user, and permission state.
- **Authentication:** Auth state with `User | null`.
- **Real-time features:** WebSocket-fed state in Redux or Zustand.

### References

- Redux Toolkit – TypeScript Quick Start - https://redux-toolkit.js.org/tutorials/typescript
- Redux Toolkit – `createSlice` - https://redux-toolkit.js.org/api/createSlice
- Redux Toolkit – `createAsyncThunk` - https://redux-toolkit.js.org/api/createAsyncThunk
- Redux – Usage with TypeScript - https://redux.js.org/usage/usage-with-typescript
- Zustand – TypeScript Guide - https://zustand.docs.pmnd.rs/guides/typescript
- Zustand – `useShallow` - https://zustand.docs.pmnd.rs/hooks/use-shallow
- React Official Documentation – Scaling Up with Reducer and Context - https://react.dev/learn/scaling-up-with-reducer-and-context
- TanStack Query – TypeScript - https://tanstack.com/query/latest/docs/framework/react/typescript

---

## Core Concept 3: Form Types & Dynamic Inputs

### Definitions

**Core Definition:** Form types and dynamic inputs are the TypeScript types for a form's values, errors, validation rules, and dynamically generated fields, integrated with a form library such as React Hook Form.

**Technical Definition:** React Hook Form types forms via `useForm<FormValues>({ resolver })`, where `FormValues` is the shape of the form data. The `register` function is typed to accept only keys of `FormValues`, and the `handleSubmit` callback receives `FormValues`. Validation is provided by a resolver (`zodResolver(schema)`), which infers the form's type from the schema. Dynamic inputs—fields that appear based on user choices—are modelled as discriminated unions, and `useFieldArray` handles arrays of repeating fields with typed `fields` and `append`/`remove` operations. Multi-step forms keep all steps' values in a single `FormValues` type and validate per-step with `trigger`. `formState.errors` is typed as `FieldErrors<FormValues>`, providing type-safe error access. The `Controller` component types custom inputs via `Controller<FormValues, 'fieldName'>`.

**Beginner-Friendly Explanation:** A form's type is the shape of its data—for example, `{ name: string; email: string; age: number }`. React Hook Form uses this type to check that you register the right field names, pass the right values, and read the right errors. For dynamic forms (add/remove rows, conditional fields), you use discriminated unions and `useFieldArray`. The result is that a typo in a field name is a compile error, not a runtime bug.

### Purposes

- To type the form's values, errors, and validation rules.
- To enforce that `register` is called with valid field names.
- To type the `handleSubmit` callback's data.
- To type dynamic field arrays (`useFieldArray`).
- To type multi-step forms with a single value type.
- To type `Controller` for custom input components.
- To infer form types from validation schemas (Zod).

### Syntax Rules and Structure

**Basic Form with Zod:**
```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const loginSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'At least 8 characters'),
});

type LoginForm = z.infer<typeof loginSchema>;

function LoginPage() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<LoginForm>({
    resolver: zodResolver(loginSchema),
    defaultValues: { email: '', password: '' },
  });

  const onSubmit = (data: LoginForm) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span role="alert">{errors.email.message}</span>}
      <input type="password" {...register('password')} />
      {errors.password && <span role="alert">{errors.password.message}</span>}
      <button type="submit">Log In</button>
    </form>
  );
}
```

**Component Breakdown:**
- `z.infer<typeof loginSchema>`: Infers the `LoginForm` type from the schema.
- `useForm<LoginForm>`: Types the form's values.
- `register('email')`: TypeScript enforces that `email` is a key of `LoginForm`.
- `errors.email`: Typed as `FieldError | undefined`.

**Dynamic Field Array (`useFieldArray`):**
```tsx
import { useForm, useFieldArray } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.object({
  phoneNumbers: z.array(
    z.object({
      type: z.enum(['mobile', 'home', 'work']),
      number: z.string().min(1, 'Required'),
    })
  ).min(1, 'At least one phone number'),
});

type FormValues = z.infer<typeof schema>;

function PhoneForm() {
  const { register, control, handleSubmit, formState: { errors } } = useForm<FormValues>({
    resolver: zodResolver(schema),
    defaultValues: { phoneNumbers: [{ type: 'mobile', number: '' }] },
  });

  const { fields, append, remove } = useFieldArray({ control, name: 'phoneNumbers' });

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      {fields.map((field, index) => (
        <div key={field.id}>
          <select {...register(`phoneNumbers.${index}.type`)}>
            <option value="mobile">Mobile</option>
            <option value="home">Home</option>
            <option value="work">Work</option>
          </select>
          <input {...register(`phoneNumbers.${index}.number`)} />
          {errors.phoneNumbers?.[index]?.number && (
            <span role="alert">{errors.phoneNumbers[index]?.number?.message}</span>
          )}
          <button type="button" onClick={() => remove(index)}>Remove</button>
        </div>
      ))}
      <button type="button" onClick={() => append({ type: 'mobile', number: '' })}>Add Phone</button>
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Component Breakdown:**
- `z.array(z.object({ ... }))`: The array's element type.
- `useFieldArray({ control, name: 'phoneNumbers' })`: Types `fields` as `{ id: string; type: 'mobile' | 'home' | 'work'; number: string }[]`.
- `register(\`phoneNumbers.${index}.type\`)`: Template literal keys are validated.
- `errors.phoneNumbers?.[index]?.number`: Nested error access.

**Discriminated Union for Dynamic Fields:**
```tsx
import { useForm, useWatch } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.discriminatedUnion('accountType', [
  z.object({
    accountType: z.literal('individual'),
    firstName: z.string().min(1),
    lastName: z.string().min(1),
  }),
  z.object({
    accountType: z.literal('business'),
    companyName: z.string().min(1),
    taxId: z.string().regex(/^\d{9}$/),
  }),
]);

type FormValues = z.infer<typeof schema>;

function RegistrationForm() {
  const { register, control, handleSubmit, formState: { errors } } = useForm<FormValues>({
    resolver: zodResolver(schema),
    defaultValues: { accountType: 'individual', firstName: '', lastName: '' },
  });

  const accountType = useWatch({ control, name: 'accountType' });

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      <select {...register('accountType')}>
        <option value="individual">Individual</option>
        <option value="business">Business</option>
      </select>

      {accountType === 'individual' && (
        <>
          <input {...register('firstName')} placeholder="First Name" />
          {errors.firstName && <span role="alert">{errors.firstName.message}</span>}
          <input {...register('lastName')} placeholder="Last Name" />
        </>
      )}

      {accountType === 'business' && (
        <>
          <input {...register('companyName')} placeholder="Company Name" />
          <input {...register('taxId')} placeholder="Tax ID" />
        </>
      )}

      <button type="submit">Register</button>
    </form>
  );
}
```

**Component Breakdown:**
- `z.discriminatedUnion('accountType', [...])`: The form's type changes based on `accountType`.
- `useWatch({ control, name: 'accountType' })`: Watches the discriminant.
- `register('firstName')` is valid only when `accountType === 'individual'`; TypeScript narrows the form type.

**Multi-Step Form:**
```tsx
import { useForm, FormProvider, useFormContext } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const fullSchema = z.object({
  firstName: z.string().min(1),
  lastName: z.string().min(1),
  email: z.string().email(),
  street: z.string().min(1),
  city: z.string().min(1),
  zip: z.string().regex(/^\d{5}$/),
});

type FormValues = z.infer<typeof fullSchema>;

const steps: Array<{ fields: (keyof FormValues)[]; label: string }> = [
  { fields: ['firstName', 'lastName', 'email'], label: 'Personal' },
  { fields: ['street', 'city', 'zip'], label: 'Address' },
];

function StepFields({ fields }: { fields: (keyof FormValues)[] }) {
  const { register, formState: { errors } } = useFormContext<FormValues>();
  return (
    <>
      {fields.map((field) => (
        <div key={field}>
          <label htmlFor={field}>{field}</label>
          <input id={field} {...register(field)} />
          {errors[field] && <span role="alert">{errors[field]?.message}</span>}
        </div>
      ))}
    </>
  );
}

function Wizard() {
  const [step, setStep] = React.useState(0);
  const methods = useForm<FormValues>({
    resolver: zodResolver(fullSchema),
    defaultValues: { firstName: '', lastName: '', email: '', street: '', city: '', zip: '' },
  });

  async function next() {
    const valid = await methods.trigger(steps[step].fields);
    if (valid) setStep((s) => s + 1);
  }

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit((data) => console.log(data))}>
        <StepFields fields={steps[step].fields} />
        {step > 0 && <button type="button" onClick={() => setStep((s) => s - 1)}>Back</button>}
        {step < steps.length - 1 && <button type="button" onClick={next}>Next</button>}
        {step === steps.length - 1 && <button type="submit">Submit</button>}
      </form>
    </FormProvider>
  );
}
```

**Component Breakdown:**
- `FormValues`: The full form type.
- `steps[step].fields`: `(keyof FormValues)[]`, ensuring only valid fields are validated per step.
- `useFormContext<FormValues>()`: Types the context in child components.
- `trigger(fields)`: Validates only the current step's fields.

**Syntax Rules:**
- Define the form's type from a Zod schema (`z.infer`) or explicitly.
- Pass the type to `useForm<FormValues>()`.
- Use `zodResolver(schema)` for validation; the resolver infers the schema's type.
- Use `useFieldArray` for arrays; `fields` are typed, and `register` uses template literal keys.
- Use discriminated unions for dynamic fields based on a discriminant.
- Use `FormProvider` and `useFormContext<FormValues>()` for multi-step forms.
- Use `trigger(fields)` to validate only the current step's fields.
- Type `Controller` as `Controller<FormValues, 'fieldName'>` for custom inputs.

**Constraints and Limitations:**
- `register` with a dynamically constructed key (e.g., `\`items.${i}.name\``) requires template literal types; TypeScript validates the path if the array type is known.
- `useFieldArray` `fields` include an `id` from the library; do not use it as a domain ID.
- `zodResolver` requires `@hookform/resolvers` and matching Zod versions.
- Multi-step forms must keep all fields mounted (or use `shouldUnregister: false`) to preserve values.
- Discriminated unions in forms require `useWatch` or `watch` to render conditionally.
- Errors for nested fields are typed as `FieldErrors<FormValues>` with optional chaining.

### Annotated Code Examples

**Example 1: Multi-Step Form with Typed Steps**

```tsx
type StepConfig = {
  label: string;
  fields: (keyof FormValues)[];
};

const steps: StepConfig[] = [
  { label: 'Personal', fields: ['firstName', 'lastName', 'email'] },
  { label: 'Address', fields: ['street', 'city', 'zip'] },
  { label: 'Review', fields: [] },
];

function Wizard() {
  const [step, setStep] = React.useState(0);
  const methods = useForm<FormValues>({
    resolver: zodResolver(fullSchema),
    mode: 'onBlur',
    defaultValues: { firstName: '', lastName: '', email: '', street: '', city: '', zip: '' },
  });

  async function next() {
    const valid = await methods.trigger(steps[step].fields);
    if (valid) setStep((s) => Math.min(s + 1, steps.length - 1));
  }

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit((data) => console.log(data))}>
        <ol>
          {steps.map((s, i) => (
            <li key={s.label} aria-current={i === step ? 'step' : undefined}>{s.label}</li>
          ))}
        </ol>
        {step < steps.length - 1 ? (
          <StepFields fields={steps[step].fields} />
        ) : (
          <ReviewStep />
        )}
        {step > 0 && <button type="button" onClick={() => setStep((s) => s - 1)}>Back</button>}
        {step < steps.length - 1 && <button type="button" onClick={next}>Next</button>}
        {step === steps.length - 1 && <button type="submit">Submit</button>}
      </form>
    </FormProvider>
  );
}
```

**Expected Output:** A three-step wizard with Personal, Address, and Review steps. The `Next` button validates only the current step's fields. The `fields` array is typed as `(keyof FormValues)[]`, so TypeScript catches typos like `'firstname'`.

**Why This Output Occurs:** `StepConfig.fields` is `(keyof FormValues)[]`, restricting the field names to valid keys. `trigger(fields)` validates only those fields. `useFormContext<FormValues>()` types the child components.

**Example 2: Dynamic Fields with Discriminated Union**

```tsx
function RegistrationForm() {
  const methods = useForm<FormValues>({
    resolver: zodResolver(schema),
    defaultValues: { accountType: 'individual', firstName: '', lastName: '' },
  });

  const accountType = useWatch({ control: methods.control, name: 'accountType' });

  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit((data) => console.log(data))}>
        <select {...methods.register('accountType')}>
          <option value="individual">Individual</option>
          <option value="business">Business</option>
        </select>

        {accountType === 'individual' && (
          <>
            <input {...methods.register('firstName')} />
            <input {...methods.register('lastName')} />
          </>
        )}

        {accountType === 'business' && (
          <>
            <input {...methods.register('companyName')} />
            <input {...methods.register('taxId')} />
          </>
        )}

        <button type="submit">Register</button>
      </form>
    </FormProvider>
  );
}
```

**Expected Output:** The form renders different fields based on `accountType`. TypeScript narrows the form type: `register('firstName')` is valid only when `accountType === 'individual'`; `register('companyName')` is valid only when `accountType === 'business'`.

**Why This Output Occurs:** The Zod schema uses `z.discriminatedUnion('accountType', [...])`. `useWatch` narrows `accountType` to `'individual' | 'business'`, and TypeScript narrows the form type based on the branch.

### Real-World Cases

- **Signup/login forms:** Typed email/password fields with validation.
- **Checkout forms:** Multi-step address, shipping, and payment.
- **Dynamic surveys:** Add/remove questions with typed field arrays.
- **Invoice builders:** Line items with typed product references.
- **Profile editors:** Conditional fields based on account type.
- **Admin CRUD forms:** Create/edit modes with discriminated unions.

### References

- React Hook Form – TypeScript - https://react-hook-form.com/ts
- React Hook Form – `useForm` - https://react-hook-form.com/docs/useform
- React Hook Form – `useFieldArray` - https://react-hook-form.com/docs/usefieldarray
- React Hook Form – `FormProvider` - https://react-hook-form.com/docs/formprovider
- React Hook Form – `Controller` - https://react-hook-form.com/docs/usecontroller/controller
- Zod – Documentation - https://zod.dev/
- Zod – Discriminated Unions - https://zod.dev/?id=discriminated-unions
- @hookform/resolvers – GitHub - https://github.com/react-hook-form/resolvers

---

## Core Concept 4: API Contracts & Route Parameters

### Definitions

**Core Definition:** API contracts are the typed request and response shapes for backend endpoints; route parameters are the typed URL path segments and query strings extracted by a router.

**Technical Definition:** API contracts define the request body, query parameters, path parameters, and response envelope for each endpoint. They are typically defined as Zod schemas (validated at the boundary) or TypeScript types (trusted from a generated client). A standard envelope is `ApiResponse<T> = { data: T; status: number; message?: string }` or a discriminated union `ApiResult<T> = { status: 'success'; data: T } | { status: 'error'; error: ApiError }`. Route parameters are typed via React Router's `useParams<Params>()` and `useSearchParams()`; `Params` is a type with string keys and string values. Query strings are parsed from `URLSearchParams` and validated with Zod. Type-safe routing in Next.js or React Router v7 uses generated types from the route definitions. The architectural principle is that URL and API contracts are **boundaries**: validate at the boundary, then trust the validated type throughout the app.

**Beginner-Friendly Explanation:** When your app calls an API, the server returns data of a certain shape. When your app navigates to `/users/42`, the URL contains a parameter `42`. API contracts and route parameters are the types that describe those shapes. You validate the data at the boundary (with Zod, for example), then pass it through the app knowing it is correct. This prevents bugs like `user.nmae` (typo) or `parseInt(params.userId)` failing because `userId` is undefined.

### Purposes

- To type API request bodies, query parameters, and responses.
- To validate API responses at the boundary with runtime schemas.
- To type URL path parameters (`useParams`).
- To type query strings (`useSearchParams`).
- To model standard error structures (`ApiError`).
- To provide a single source of truth for API contracts.

### Syntax Rules and Structure

**Standard API Envelope:**
```typescript
// src/types/api.ts
export type ApiResponse<T> = {
  data: T;
  status: number;
  message?: string;
};

export type ApiError = {
  code: string;
  message: string;
  details?: Record<string, string[]>;
};

export type ApiResult<T> =
  | { status: 'success'; data: T }
  | { status: 'error'; error: ApiError };

export type Paginated<T> = {
  data: T[];
  meta: {
    total: number;
    page: number;
    pageSize: number;
    nextCursor?: string;
  };
};
```

**Typed Fetcher with Validation:**
```typescript
import { z } from 'zod';
import type { ApiResponse } from '@/types/api';

const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
});

type User = z.infer<typeof UserSchema>;

async function fetchUser(id: number): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  const json = (await res.json()) as ApiResponse<unknown>;
  return UserSchema.parse(json.data);
}
```

**Typed URL Path Parameters:**
```tsx
import { useParams } from 'react-router';

type UserRouteParams = {
  userId: string;
};

function UserProfile() {
  const { userId } = useParams<UserRouteParams>();
  // userId: string | undefined
  if (!userId) return <p>No user ID</p>;
  return <UserDetail userId={Number(userId)} />;
}
```

**Typed Query Strings with Zod:**
```tsx
import { useSearchParams } from 'react-router';
import { z } from 'zod';

const SearchParamsSchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  sort: z.enum(['name', 'price', 'date']).default('name'),
  query: z.string().optional(),
});

type SearchParams = z.infer<typeof SearchParamsSchema>;

function useTypedSearchParams(): SearchParams {
  const [searchParams] = useSearchParams();
  return SearchParamsSchema.parse(Object.fromEntries(searchParams));
}

function ProductList() {
  const { page, sort, query } = useTypedSearchParams();
  // page: number, sort: 'name' | 'price' | 'date', query: string | undefined
  return <div>Page {page}, Sort {sort}, Query {query ?? 'none'}</div>;
}
```

**Component Breakdown:**
- `SearchParamsSchema`: Validates and coerces query parameters.
- `z.coerce.number()`: Converts the string `"1"` to the number `1`.
- `Object.fromEntries(searchParams)`: Converts `URLSearchParams` to a plain object.
- `parse(...)`: Throws if the parameters are invalid.

**Typed API Client with Endpoint Contracts:**
```typescript
type Endpoints = {
  'GET /users': { response: User[]; query: { page?: number } };
  'GET /users/:id': { response: User; params: { id: number } };
  'POST /users': { response: User; body: { name: string; email: string } };
};

async function call<K extends keyof Endpoints>(
  endpoint: K,
  options: Omit<Endpoints[K], 'response'>
): Promise<Endpoints[K]['response']> {
  const [method, path] = endpoint.split(' ');
  const res = await fetch(path, { method });
  return res.json() as Promise<Endpoints[K]['response']>;
}
```

**Component Breakdown:**
- `Endpoints`: A map of endpoint contracts to request and response types.
- `call<K>`: A generic function that types the response based on the endpoint.

**Syntax Rules:**
- Define a standard `ApiResponse<T>` envelope or `ApiResult<T>` discriminated union.
- Define `ApiError` with `code`, `message`, and optional `details`.
- Validate API responses with Zod at the boundary; never trust `as`.
- Type `useParams<Params>()` with the route's parameter names.
- Validate `useSearchParams()` with a Zod schema that coerces strings to numbers and enums.
- Provide defaults for optional query parameters via `.default(...)`.
- Use `z.coerce.number()` and `z.coerce.boolean()` for query string coercion.
- Keep API contracts in a shared module (`types/api.ts`, `api/contracts.ts`).
- Generate types from OpenAPI or GraphQL when possible.

**Constraints and Limitations:**
- `useParams` returns `string | undefined` for every key; always guard against `undefined`.
- `useSearchParams` returns `URLSearchParams`, which is string-only; coercion is required.
- `z.coerce.boolean()` treats `"false"` as `true` (only `"0"` and `""` are false); use an enum for booleans.
- API contracts must be kept in sync with the backend; generated types are more reliable than manual ones.
- Error structures vary by backend; normalise them at the boundary.
- Route parameters typed with `useParams<T>` are not validated at runtime; combine with Zod.

### Annotated Code Examples

**Example 1: Typed API Client with Endpoint Contracts**

```typescript
// src/api/contracts.ts
import type { User, CreateUserInput } from '@/domain';

export type Endpoints = {
  'GET /users': {
    response: User[];
    query: { page?: number; search?: string };
  };
  'GET /users/:id': {
    response: User;
    params: { id: number };
  };
  'POST /users': {
    response: User;
    body: CreateUserInput;
  };
  'DELETE /users/:id': {
    response: void;
    params: { id: number };
  };
};

export type EndpointKey = keyof Endpoints;
export type ResponseOf<K extends EndpointKey> = Endpoints[K]['response'];
```

```typescript
// src/api/client.ts
import type { Endpoints, EndpointKey, ResponseOf } from './contracts';

async function request<K extends EndpointKey>(
  endpoint: K,
  options: Omit<Endpoints[K], 'response'>
): Promise<ResponseOf<K>> {
  const [method, path] = endpoint.split(' ');
  const res = await fetch(path, {
    method,
    body: 'body' in options ? JSON.stringify(options.body) : undefined,
    headers: { 'Content-Type': 'application/json' },
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json() as Promise<ResponseOf<K>>;
}

// Usage — types are inferred from the endpoint
const users = await request('GET /users', { query: { page: 1 } }); // User[]
const user = await request('GET /users/:id', { params: { id: 1 } }); // User
// await request('POST /users', { body: { name: 'Alice' } }); // Error: email missing
```

**Expected Output:** The `request` function is generic over the endpoint key. TypeScript infers the response type from the endpoint (`User[]` for `GET /users`, `User` for `GET /users/:id`). Passing an invalid body errors at compile time.

**Why This Output Occurs:** `Endpoints` maps each endpoint to its request and response types. `request<K>` returns `ResponseOf<K>`, so the response type is inferred from the endpoint. The `options` parameter is typed as `Omit<Endpoints[K], 'response'>`, so the correct request shape is enforced.

**Example 2: Typed Query String Parsing**

```tsx
import { useSearchParams } from 'react-router';
import { z } from 'zod';

const FiltersSchema = z.object({
  page: z.coerce.number().int().min(1).catch(1),
  perPage: z.coerce.number().int().min(1).max(100).catch(20),
  sort: z.enum(['name', 'price', 'date']).catch('name'),
  order: z.enum(['asc', 'desc']).catch('asc'),
  query: z.string().optional(),
  inStock: z.coerce.boolean().catch(false),
});

type Filters = z.infer<typeof FiltersSchema>;

function useFilters(): Filters {
  const [searchParams] = useSearchParams();
  return FiltersSchema.parse(Object.fromEntries(searchParams));
}

function ProductList() {
  const { page, perPage, sort, order, query, inStock } = useFilters();
  // page: number, perPage: number, sort: 'name' | 'price' | 'date',
  // order: 'asc' | 'desc', query: string | undefined, inStock: boolean
  return (
    <div>
      <p>Page {page}, {perPage} per page</p>
      <p>Sort: {sort} ({order})</p>
      <p>Query: {query ?? 'none'}, In stock: {String(inStock)}</p>
    </div>
  );
}
```

**Expected Output:** The `useFilters` hook parses `useSearchParams` through the Zod schema, coercing strings to numbers and booleans, applying defaults via `.catch(...)`, and typing the result. Invalid parameters fall back to defaults instead of throwing.

**Why This Output Occurs:** `z.coerce.number()` converts the string `"1"` to `1`. `.catch(1)` provides a fallback if parsing fails. `z.enum([...])` restricts `sort` and `order`. `z.coerce.boolean()` coerces `"true"` and `"false"` (note: only `"false"` and `""` become `false`; `"0"` becomes `true`).

### Real-World Cases

- **REST APIs:** Typed fetchers with validated responses.
- **OpenAPI:** Generated types from specs.
- **GraphQL:** Codegen-generated query and mutation types.
- **tRPC:** End-to-end type safety without codegen.
- **URL state:** Typed filters, pagination, and sort parameters.
- **Error handling:** Standard `ApiError` shape with field errors.

### References

- React Router – `useParams` - https://reactrouter.com/api/hooks/useParams
- React Router – `useSearchParams` - https://reactrouter.com/api/hooks/useSearchParams
- Zod – Documentation - https://zod.dev/
- Zod – Coercion - https://zod.dev/?id=coercion
- TanStack Query – TypeScript - https://tanstack.com/query/latest/docs/framework/react/typescript
- OpenAPI TypeScript - https://github.com/drwpow/openapi-typescript
- GraphQL Code Generator - https://the-guild.dev/graphql/codegen
- tRPC - https://trpc.io/

---

## Core Concept 5: Validation Schemas

### Definitions

**Core Definition:** Validation schemas are runtime validators (Zod, Yup, ArkType) that define the shape and constraints of data, and from which TypeScript types are inferred, providing a single source of truth for both runtime and compile-time correctness.

**Technical Definition:** A validation schema is a declarative description of a data shape and its constraints (required fields, string lengths, numeric ranges, enums, custom refinements). Libraries such as **Zod**, **Yup**, and **ArkType** parse input at runtime, returning either the validated data or an error. They also expose type inference: `z.infer<typeof Schema>` (Zod), `InferType<typeof Schema>` (Yup), and `typeof Schema.infer` (ArkType). This makes the schema the single source of truth: the schema defines both validation and the TypeScript type. Schemas can be composed (`z.object().extend()`, `.merge()`, `.pick()`, `.omit()`), refined (`.refine()`, `.superRefine()`), transformed (`.transform()`), and discriminated (`z.discriminatedUnion()`). Zod is the most popular in TypeScript-first codebases; Yup remains common with Formik; ArkType offers faster runtime parsing and a more terse syntax. The architectural principle is **schema-first**: define schemas at the boundary (API, form, URL), infer types, and share the schema between client and server.

**Beginner-Friendly Explanation:** A validation schema is a rulebook. It says "a user has a name (a string), an email (a valid email), and an age (a number between 13 and 120)." Zod reads the rulebook and both validates the data at runtime and produces a TypeScript type. So you define the rules once, and both the runtime check and the compile-time type come from the same place. If the rules change, the type changes, and every place that uses it updates.

### Purposes

- To validate data at the boundary (API, form, URL) at runtime.
- To infer TypeScript types from schemas (`z.infer`).
- To share schemas between client and server.
- To compose schemas from smaller building blocks.
- To refine and transform data during validation.
- To provide detailed, path-aware error messages.

### Syntax Rules and Structure

**Zod — Basic Schema:**
```typescript
import { z } from 'zod';

const UserSchema = z.object({
  id: z.number(),
  name: z.string().min(2).max(50),
  email: z.string().email(),
  age: z.number().int().min(13).max(120),
  role: z.enum(['admin', 'user', 'guest']),
  avatar: z.string().url().optional(),
  createdAt: z.string().datetime(),
});

type User = z.infer<typeof UserSchema>;

const user = UserSchema.parse(data); // Throws on invalid
const result = UserSchema.safeParse(data); // Returns { success, data | error }
```

**Yup — Basic Schema:**
```typescript
import * as yup from 'yup';

const UserSchema = yup.object({
  id: yup.number().required(),
  name: yup.string().min(2).max(50).required(),
  email: yup.string().email().required(),
  age: yup.number().integer().min(13).max(120).required(),
  role: yup.mixed<'admin' | 'user' | 'guest'>().oneOf(['admin', 'user', 'guest']).required(),
});

type User = yup.InferType<typeof UserSchema>;
```

**ArkType — Basic Schema:**
```typescript
import { type } from 'arktype';

const UserSchema = type({
  id: 'number',
  name: 'string >= 2 <= 50',
  email: 'string.email',
  age: 'number.integer >= 13 <= 120',
  role: '"admin" | "user" | "guest"',
});

type User = typeof UserSchema.infer;
```

**Schema Composition (Zod):**
```typescript
const BaseUserSchema = z.object({
  name: z.string(),
  email: z.string().email(),
});

const CreateUserSchema = BaseUserSchema.extend({
  password: z.string().min(8),
});

const UpdateUserSchema = BaseUserSchema.partial();

const UserResponseSchema = BaseUserSchema.extend({
  id: z.number(),
  createdAt: z.string().datetime(),
});

type CreateUser = z.infer<typeof CreateUserSchema>;
type UpdateUser = z.infer<typeof UpdateUserSchema>;
type UserResponse = z.infer<typeof UserResponseSchema>;
```

**Refinements and Transformations (Zod):**
```typescript
const PasswordSchema = z.string()
  .min(8, 'At least 8 characters')
  .regex(/[A-Z]/, 'Must contain an uppercase letter')
  .regex(/\d/, 'Must contain a number');

const RegistrationSchema = z.object({
  password: PasswordSchema,
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword'],
});

// Transformation
const DateSchema = z.string().datetime().transform((s) => new Date(s));
type DateType = z.infer<typeof DateSchema>; // Date
```

**Discriminated Union (Zod):**
```typescript
const PaymentSchema = z.discriminatedUnion('method', [
  z.object({ method: z.literal('card'), cardNumber: z.string().regex(/^\d{16}$/) }),
  z.object({ method: z.literal('bank'), accountNumber: z.string() }),
  z.object({ method: z.literal('paypal'), email: z.string().email() }),
]);

type Payment = z.infer<typeof PaymentSchema>;
```

**Sharing Schemas Between Client and Server:**
```typescript
// shared/schemas/user.ts — imported by both client and server
export const CreateUserSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  password: z.string().min(8),
});

export type CreateUserInput = z.infer<typeof CreateUserSchema>;
```

```typescript
// server/routes/users.ts
import { CreateUserSchema } from '../../shared/schemas/user';

export async function createUser(req: Request) {
  const result = CreateUserSchema.safeParse(await req.json());
  if (!result.success) {
    return Response.json({ errors: result.error.flatten().fieldErrors }, { status: 422 });
  }
  const user = await db.users.create(result.data);
  return Response.json({ data: user });
}
```

**Syntax Rules:**
- Define schemas with `z.object({ ... })`, `yup.object({ ... })`, or `type({ ... })`.
- Infer types with `z.infer<typeof Schema>`, `yup.InferType<typeof Schema>`, or `typeof Schema.infer`.
- Use `.parse()` for throwing validation and `.safeParse()` for non-throwing.
- Use `.extend()`, `.merge()`, `.pick()`, `.omit()`, `.partial()` to compose schemas.
- Use `.refine()` and `.superRefine()` for custom cross-field validation.
- Use `.transform()` to convert data during validation (e.g., string to `Date`).
- Use `z.discriminatedUnion('discriminant', [...])` for mutually exclusive shapes.
- Share schemas between client and server by extracting them into a shared module.
- Use `.catch()` for defaults that should apply when parsing fails (e.g., URL params).

**Constraints and Limitations:**
- Runtime validation adds a small performance cost; measure in hot paths.
- Zod's bundle size is significant (~12 KB gzipped); Yup is ~15 KB; ArkType is smaller but less mature.
- `.transform()` changes the output type; `z.input` gives the input type, `z.output` the output.
- Yup's TypeScript inference is less precise than Zod's for complex schemas.
- ArkType's syntax is concise but has a steeper learning curve.
- Schemas must be kept in sync across client and server; shared modules are the solution.
- Deeply nested refinements can be expensive; keep them shallow.

### Annotated Code Examples

**Example 1: Shared Schema Between Client and Server**

```typescript
// shared/schemas/registration.ts
import { z } from 'zod';

export const RegistrationSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword'],
});

export type RegistrationInput = z.infer<typeof RegistrationSchema>;
```

```tsx
// client/RegistrationForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { RegistrationSchema, type RegistrationInput } from '@/shared/schemas/registration';

function RegistrationForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<RegistrationInput>({
    resolver: zodResolver(RegistrationSchema),
    defaultValues: { name: '', email: '', password: '', confirmPassword: '' },
  });

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      <input {...register('name')} />
      {errors.name && <span role="alert">{errors.name.message}</span>}
      <input {...register('email')} />
      {errors.email && <span role="alert">{errors.email.message}</span>}
      <input type="password" {...register('password')} />
      {errors.password && <span role="alert">{errors.password.message}</span>}
      <input type="password" {...register('confirmPassword')} />
      {errors.confirmPassword && <span role="alert">{errors.confirmPassword.message}</span>}
      <button type="submit">Register</button>
    </form>
  );
}
```

```typescript
// server/routes/register.ts
import { RegistrationSchema } from '@/shared/schemas/registration';

export async function register(req: Request) {
  const result = RegistrationSchema.safeParse(await req.json());
  if (!result.success) {
    return Response.json({ errors: result.error.flatten().fieldErrors }, { status: 422 });
  }
  const user = await db.users.create(result.data);
  return Response.json({ data: user }, { status: 201 });
}
```

**Expected Output:** The client form validates with the shared schema; the server validates with the same schema. If the passwords do not match, both client and server return the same error message. The `RegistrationInput` type is inferred from the schema and used by `useForm`.

**Why This Output Occurs:** The `RegistrationSchema` is defined once and imported by both client and server. `z.infer` produces `RegistrationInput`, which types the form. `safeParse` on the server validates the payload and returns field errors.

**Example 2: Schema Composition and Refinement**

```typescript
const AddressSchema = z.object({
  street: z.string().min(1),
  city: z.string().min(1),
  zip: z.string().regex(/^\d{5}$/, 'Invalid ZIP'),
  country: z.enum(['US', 'CA', 'UK']),
});

const BaseProductSchema = z.object({
  name: z.string().min(1),
  price: z.number().positive(),
  sku: z.string().regex(/^[A-Z0-9-]+$/),
});

const PhysicalProductSchema = BaseProductSchema.extend({
  type: z.literal('physical'),
  weight: z.number().positive(),
  dimensions: z.object({
    width: z.number().positive(),
    height: z.number().positive(),
    depth: z.number().positive(),
  }),
});

const DigitalProductSchema = BaseProductSchema.extend({
  type: z.literal('digital'),
  downloadUrl: z.string().url(),
  fileSize: z.number().positive(),
});

const ProductSchema = z.discriminatedUnion('type', [
  PhysicalProductSchema,
  DigitalProductSchema,
]);

type Product = z.infer<typeof ProductSchema>;

const OrderSchema = z.object({
  id: z.string().uuid(),
  items: z.array(z.object({
    product: ProductSchema,
    quantity: z.number().int().positive(),
  })).min(1),
  shippingAddress: AddressSchema,
  billingAddress: AddressSchema.optional(),
  total: z.number().positive(),
}).refine((order) => {
  const computed = order.items.reduce((sum, i) => sum + i.product.price * i.quantity, 0);
  return Math.abs(computed - order.total) < 0.01;
}, { message: 'Total does not match item prices', path: ['total'] });

type Order = z.infer<typeof OrderSchema>;
```

**Expected Output:** The `OrderSchema` composes `ProductSchema` (itself a discriminated union of physical and digital products) with `AddressSchema` and refines the total against the item prices. TypeScript infers the full `Order` type, including the union of product types.

**Why This Output Occurs:** `BaseProductSchema.extend()` creates physical and digital variants. `z.discriminatedUnion('type', [...])` combines them. `OrderSchema` composes them with array, address, and refinement. The refinement checks the computed total.

### Real-World Cases

- **API validation:** Validating request bodies and responses.
- **Form validation:** React Hook Form + Zod resolver.
- **URL state:** Parsing and validating query strings.
- **Config validation:** Validating environment variables at startup.
- **Data pipelines:** Validating data before processing.
- **Shared contracts:** Client and server using the same schema.

### References

- Zod – Documentation - https://zod.dev/
- Zod – Type Inference - https://zod.dev/?id=infer-type
- Zod – Refinements - https://zod.dev/?id=refine
- Zod – Discriminated Unions - https://zod.dev/?id=discriminated-unions
- Zod – Transformations - https://zod.dev/?id=transform
- Yup – Documentation - https://github.com/jquense/yup
- ArkType – Documentation - https://arktype.io/
- @hookform/resolvers – GitHub - https://github.com/react-hook-form/resolvers
- TanStack Form – Zod Validation - https://tanstack.com/form/latest/docs/framework/react/guides/validation

---

## Core Concept 6: Ecosystem Integration

### Definitions

**Core Definition:** Ecosystem integration is the practice of handling TypeScript types from third-party libraries—including declaration merging, module augmentation, and overriding missing or incorrect types—so that the application's types remain correct and complete.

**Technical Definition:** Third-party libraries ship their own TypeScript types, but these types may be incomplete, incorrect, or missing entirely. Integration strategies include: (1) **Using the library's types as-is** when they are correct; (2) **Module augmentation** (`declare module 'library' {}`) to add missing properties or overloads; (3) **Declaration merging** to extend interfaces the library exports; (4) **Wrapper components and hooks** that add type safety on top of untyped libraries; (5) **`@types/*` packages** for libraries that do not ship types; (6) **Local `.d.ts` overrides** for libraries with incorrect types; and (7) **`skipLibCheck: false`** to catch issues in dependencies (or `true` to speed up compilation). The architectural principle is to isolate third-party type fixes in a dedicated `types/` directory, document why each fix exists, and upgrade the library (and remove the fix) when the library fixes the issue upstream. Common targets include `react-router` (augmenting `FutureConfig`), `styled-components` (augmenting `DefaultTheme`), and environment variables (`ProcessEnv`).

**Beginner-Friendly Explanation:** Third-party libraries sometimes have TypeScript types that are missing a property, wrong, or absent entirely. Ecosystem integration is how you fix that without patching the library. You can augment the module's types (add the missing property), merge with the library's interfaces (extend its types), or wrap the library in a typed helper. You keep all these fixes in one folder (`types/`) so you can remove them when the library fixes the issue.

### Purposes

- To fix missing or incorrect types in third-party libraries.
- To add custom properties to library interfaces (e.g., `DefaultTheme`).
- To augment `Window`, `ProcessEnv`, and other globals.
- To wrap untyped libraries with typed interfaces.
- To isolate third-party fixes in a dedicated directory.
- To document why each type fix exists.

### Syntax Rules and Structure

**Module Augmentation (Adding Missing Properties):**
```typescript
// src/types/react-router.d.ts
import 'react-router';

declare module 'react-router' {
  interface FutureConfig {
    v7_partialHydration?: boolean;
    v7_normalizeFormMethod?: boolean;
  }
}
```

**Declaration Merging (Extending Library Interfaces):**
```typescript
// src/types/styled-components.d.ts
import 'styled-components';

declare module 'styled-components' {
  export interface DefaultTheme {
    colors: {
      primary: string;
      secondary: string;
      background: string;
      text: string;
    };
    spacing: (units: number) => string;
    radii: {
      sm: string;
      md: string;
      lg: string;
    };
  }
}
```

**Adding Types for an Untyped Library:**
```typescript
// src/types/untyped-library.d.ts
declare module 'untyped-library' {
  export type Config = {
    apiKey: string;
    region: 'us' | 'eu' | 'asia';
  };

  export class Client {
    constructor(config: Config);
    query(sql: string): Promise<unknown[]>;
    close(): void;
  }

  export function createClient(config: Config): Client;
}
```

**Wrapper for an Untyped Library:**
```typescript
// src/lib/analytics.ts
type TrackEvent =
  | { name: 'page_view'; path: string }
  | { name: 'button_click'; label: string }
  | { name: 'purchase'; orderId: string; total: number };

// Wraps an untyped third-party analytics library
export function track(event: TrackEvent): void {
  const payload = JSON.stringify(event);
  window.__ANALYTICS__?.track(event.name, { payload });
}

// Usage — discriminated union catches invalid events
track({ name: 'button_click', label: 'cta' });
// track({ name: 'purchase', orderId: '123' }); // Error: total missing
```

**Overriding Incorrect Types with `as` at the Boundary:**
```typescript
// src/lib/third-party.ts
import type { ThirdPartyOptions } from 'third-party';

// The library's type is incorrect; we override at the boundary
const correctOptions = {
  timeout: 5000,
  retries: 3,
} satisfies ThirdPartyOptions as ThirdPartyOptions;

// Or create a typed wrapper
export function createClient(options: ThirdPartyOptions): ThirdPartyClient {
  return new ThirdPartyClient(options) as ThirdPartyClient;
}
```

**Environment Variable Typing:**
```typescript
// src/types/env.d.ts
declare namespace NodeJS {
  interface ProcessEnv {
    readonly NODE_ENV: 'development' | 'production' | 'test';
    readonly NEXT_PUBLIC_API_URL: string;
    readonly NEXT_PUBLIC_SENTRY_DSN?: string;
    readonly DATABASE_URL: string;
  }
}
```

**React Query Type Registration:**
```typescript
// src/types/react-query.d.ts
import '@tanstack/react-query';

declare module '@tanstack/react-query' {
  interface Register {
    defaultError: Error;
    queryKey: readonly unknown[];
  }
}
```

**Syntax Rules:**
- Place third-party type fixes in a dedicated `types/` directory.
- Use `import 'library'` before `declare module 'library'` to ensure the module is loaded.
- Use `declare module 'library' { interface X { ... } }` to merge interfaces.
- Use `declare module 'untyped-library' { ... }` to type an untyped library.
- Wrap untyped libraries in typed helpers (`src/lib/`) that expose a clean, narrow API.
- Document each type fix with a comment explaining why it exists.
- Remove type fixes when the library fixes the issue upstream; pin the library version to avoid breakage.
- Use `skipLibCheck: true` in `tsconfig.json` to avoid type errors in dependencies, unless you are auditing them.

**Constraints and Limitations:**
- Module augmentation requires the augmented module to have the same shape; incorrect augmentation silently fails.
- Declaration merging cannot change existing property types; it can only add new ones.
- `@types/*` packages may be out of date; prefer libraries that ship their own types.
- Wrapping untyped libraries adds boilerplate but provides type safety.
- `skipLibCheck: true` hides type errors in dependencies; use it to speed up compilation, not to ignore problems.
- Type fixes can break when the library updates; pin versions and review changes.

### Annotated Code Examples

**Example 1: Augmenting `styled-components` `DefaultTheme`**

```typescript
// src/types/styled-components.d.ts
import 'styled-components';

declare module 'styled-components' {
  export interface DefaultTheme {
    colors: {
      primary: string;
      secondary: string;
      background: string;
      text: string;
      border: string;
    };
    space: {
      xs: string;
      sm: string;
      md: string;
      lg: string;
      xl: string;
    };
    radii: {
      sm: string;
      md: string;
      lg: string;
      full: string;
    };
  }
}
```

```tsx
// src/theme.ts
import type { DefaultTheme } from 'styled-components';

export const theme: DefaultTheme = {
  colors: {
    primary: '#007bff',
    secondary: '#6c757d',
    background: '#ffffff',
    text: '#1a1a1a',
    border: '#dee2e6',
  },
  space: { xs: '4px', sm: '8px', md: '16px', lg: '24px', xl: '32px' },
  radii: { sm: '4px', md: '8px', lg: '12px', full: '9999px' },
};
```

```tsx
// src/components/Button.tsx
import styled from 'styled-components';

export const Button = styled.button`
  background: ${({ theme }) => theme.colors.primary};
  padding: ${({ theme }) => theme.space.md} ${({ theme }) => theme.space.lg};
  border-radius: ${({ theme }) => theme.radii.md};
  color: white;
`;
```

**Expected Output:** The `theme` object is typed as `DefaultTheme`, so TypeScript enforces that all properties (`colors`, `space`, `radii`) are present and correctly typed. The `Button` component accesses `theme.colors.primary`, `theme.space.md`, and `theme.radii.md` with full autocomplete.

**Why This Output Occurs:** The `declare module 'styled-components'` block merges the `DefaultTheme` interface with the custom shape. All consumers of `DefaultTheme` (including the `theme` prop in styled-components) see the augmented type.

**Example 2: Wrapping an Untyped Library**

```typescript
// src/types/chartjs.d.ts
declare module 'chartjs-unofficial' {
  export type ChartConfig = {
    type: 'line' | 'bar' | 'pie';
    data: { labels: string[]; datasets: Array<{ data: number[]; label: string }> };
    options?: Record<string, unknown>;
  };

  export class Chart {
    constructor(canvas: HTMLCanvasElement, config: ChartConfig);
    update(): void;
    destroy(): void;
  }
}
```

```tsx
// src/components/LineChart.tsx
import { useEffect, useRef } from 'react';
import { Chart, type ChartConfig } from 'chartjs-unofficial';

type LineChartProps = {
  labels: string[];
  values: number[];
  title: string;
};

export function LineChart({ labels, values, title }: LineChartProps) {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const chartRef = useRef<Chart | null>(null);

  useEffect(() => {
    if (!canvasRef.current) return;

    const config: ChartConfig = {
      type: 'line',
      data: {
        labels,
        datasets: [{ data: values, label: title }],
      },
      options: { responsive: true },
    };

    chartRef.current = new Chart(canvasRef.current, config);
    return () => chartRef.current?.destroy();
  }, [labels, values, title]);

  return <canvas ref={canvasRef} />;
}
```

**Expected Output:** The `LineChart` component uses the untyped library through the local `.d.ts` declaration. TypeScript enforces that `config` matches `ChartConfig` and that `chartRef.current` is a `Chart | null`.

**Why This Output Occurs:** The `declare module 'chartjs-unofficial'` block provides types for the untyped library. The component uses those types to construct the config and manage the chart instance.

### Real-World Cases

- **`styled-components`:** Augmenting `DefaultTheme`.
- **`react-router`:** Augmenting `FutureConfig`.
- **`@tanstack/react-query`:** Registering the default error and query key types.
- **Environment variables:** Typing `process.env` and `import.meta.env`.
- **Untyped libraries:** Wrapping analytics, chart, or map libraries.
- **Global augmentation:** Adding `window.__APP_VERSION__`.

### References

- TypeScript Handbook – Module Augmentation - https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation
- TypeScript Handbook – Declaration Merging - https://www.typescriptlang.org/docs/handbook/declaration-merging.html
- TypeScript Handbook – Declaration Files - https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html
- TypeScript Handbook – `skipLibCheck` - https://www.typescriptlang.org/tsconfig#skipLibCheck
- DefinitelyTyped - https://github.com/DefinitelyTyped/DefinitelyTyped
- styled-components – TypeScript - https://styled-components.com/docs/api#typescript
- TanStack Query – TypeScript Registration - https://tanstack.com/query/latest/docs/framework/react/typescript
- React Router – TypeScript - https://reactrouter.com/explanation/type-safety

---

## Comparison and Decision Guidance

| Concept | Primary Tool | When to Use | Key Risk |
|---|---|---|---|
| **Shared types** | `domain/` module + explicit imports | All shared domain models | Type drift across layers |
| **Global declarations** | `.d.ts` with `declare global` | Library augmentation, assets, env | Global scope pollution |
| **State management** | Redux Toolkit, Zustand, Context | Client state, domain models | Duplicating server state |
| **Forms** | React Hook Form + Zod | All forms | Untyped field names |
| **Dynamic forms** | Discriminated unions + `useFieldArray` | Conditional/array fields | Errors without narrow types |
| **API contracts** | Zod schemas + `ApiResponse<T>` | All API calls | Trusting unvalidated responses |
| **Route parameters** | `useParams<T>` + Zod query schema | Typed URLs | Runtime-unvalidated params |
| **Validation schemas** | Zod (or Yup, ArkType) | Boundary validation | Schema-type drift |
| **Ecosystem integration** | `types/` + module augmentation | Third-party type fixes | Broken augmentation |

**Decision Guidance:**
- **Define domain types once** in a `domain/` module and import them explicitly.
- **Use `import type`** for type-only imports; use path aliases for consistency.
- **Use `.d.ts` files sparingly** — only for global declarations, asset modules, and library augmentation.
- **Type state with domain models** — do not redefine entities in slices or stores.
- **Derive `RootState` and `AppDispatch`** from the store; never annotate manually.
- **Use Zod schemas as the source of truth** for forms, APIs, and URL parameters.
- **Infer form types** with `z.infer<typeof schema>` and pass them to `useForm<T>`.
- **Model dynamic fields** with discriminated unions; validate per-step with `trigger`.
- **Validate API responses at the boundary** with `.parse()`; never trust `as`.
- **Type URL parameters** with `useParams<T>` and validate query strings with Zod.
- **Isolate third-party type fixes** in `types/` and document each fix.
- **Remove type fixes** when the library fixes the issue upstream.

---

## References

- TypeScript Handbook - https://www.typescriptlang.org/docs/handbook/intro.html
- TypeScript Handbook – Modules - https://www.typescriptlang.org/docs/handbook/2/modules.html
- TypeScript Handbook – Declaration Files - https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html
- TypeScript Handbook – Module Augmentation - https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation
- TypeScript Handbook – Declaration Merging - https://www.typescriptlang.org/docs/handbook/declaration-merging.html
- TypeScript Handbook – `import type` - https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html#type-only-imports-and-export
- TypeScript Handbook – Path Mapping - https://www.typescriptlang.org/tsconfig#paths
- TypeScript Handbook – `skipLibCheck` - https://www.typescriptlang.org/tsconfig#skipLibCheck
- Redux Toolkit – TypeScript Quick Start - https://redux-toolkit.js.org/tutorials/typescript
- Redux Toolkit – `createSlice` - https://redux-toolkit.js.org/api/createSlice
- Redux Toolkit – `createAsyncThunk` - https://redux-toolkit.js.org/api/createAsyncThunk
- Redux – Usage with TypeScript - https://redux.js.org/usage/usage-with-typescript
- Zustand – TypeScript Guide - https://zustand.docs.pmnd.rs/guides/typescript
- Zustand – `useShallow` - https://zustand.docs.pmnd.rs/hooks/use-shallow
- React Hook Form – TypeScript - https://react-hook-form.com/ts
- React Hook Form – `useForm` - https://react-hook-form.com/docs/useform
- React Hook Form – `useFieldArray` - https://react-hook-form.com/docs/usefieldarray
- React Hook Form – `FormProvider` - https://react-hook-form.com/docs/formprovider
- React Router – `useParams` - https://reactrouter.com/api/hooks/useParams
- React Router – `useSearchParams` - https://reactrouter.com/api/hooks/useSearchParams
- React Router – TypeScript - https://reactrouter.com/explanation/type-safety
- Zod – Documentation - https://zod.dev/
- Zod – Type Inference - https://zod.dev/?id=infer-type
- Zod – Refinements - https://zod.dev/?id=refine
- Zod – Discriminated Unions - https://zod.dev/?id=discriminated-unions
- Zod – Coercion - https://zod.dev/?id=coercion
- Yup – Documentation - https://github.com/jquense/yup
- ArkType – Documentation - https://arktype.io/
- @hookform/resolvers – GitHub - https://github.com/react-hook-form/resolvers
- TanStack Query – TypeScript - https://tanstack.com/query/latest/docs/framework/react/typescript
- OpenAPI TypeScript - https://github.com/drwpow/openapi-typescript
- GraphQL Code Generator - https://the-guild.dev/graphql/codegen
- tRPC - https://trpc.io/
- styled-components – TypeScript - https://styled-components.com/docs/api#typescript
- DefinitelyTyped - https://github.com/DefinitelyTyped/DefinitelyTyped