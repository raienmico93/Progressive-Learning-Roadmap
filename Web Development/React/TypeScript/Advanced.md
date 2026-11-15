# React Advanced TypeScript — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Advanced React TypeScript is the application of TypeScript's most powerful type-system features — generics, conditional types, mapped types, template literal types, branded types, and type-level programming — to build React components, hooks, and APIs that are simultaneously more flexible and more strictly type-safe than ordinary annotations allow.

**Technical Definition:** Advanced TypeScript exploits the **type system as a programming language**. Conditional types (`T extends U ? X : Y`) enable type-level branching; mapped types (`{ [K in keyof T]: ... }`) transform object shapes; template literal types (`` `${A}-${B}` ``) compose string literal types; `infer` extracts types from structures; and recursion at the type level enables algorithms over types. In React, these features power generic components that preserve caller types, polymorphic primitives (`as` prop), branded types for nominal typing of IDs, discriminated unions for exhaustive state machines, and compile-time tests that validate type behavior. Advanced features must be used judiciously: complexity has a cost, and **readability and inference often beat cleverness**.

**Beginner-Friendly Explanation:** Advanced TypeScript is using the type system like a mini-programming language — where types compute other types. It lets you build components that adapt to whatever data you pass in (generics), generate types from other types (mapped and conditional types), and catch entire categories of bugs at compile time (branded types, exhaustive checks). It is powerful, but you should reach for it only when simpler types are not enough.

### Key Characteristics

- **Type-level computation:** Types can branch (`extends ? :`), map over keys, recurse, and extract via `infer`.
- **Generics preserve caller types:** A well-typed generic component propagates the exact type `T` through the whole tree.
- **Nominal typing via branding:** TypeScript's structural system cannot distinguish `UserId` from `OrderId` — branded types fix that at compile time.
- **Exhaustiveness guarantees:** `never` checks turn missing `case` branches into compile errors.
- **Type testing:** `expectTypeOf` (Vitest) and `tsd`/`ts-expect` validate that types behave as intended.
- **Restraint over cleverness:** Advanced types are a liability when they produce unreadable errors or slow compilation.

### Prerequisites

- Solid command of TypeScript fundamentals: interfaces, type aliases, unions, intersections, generics, utility types, and narrowing.
- Experience typing React components, hooks, refs, and context.
- A `tsconfig.json` with `strict: true`.
- Familiarity with `keyof`, `typeof`, `infer`, `extends`, and mapped types.

### Related Programming Areas

- **React component design systems** — generic and polymorphic primitives.
- **State machines** — discriminated unions and exhaustive checking.
- **Domain modeling** — branded types for IDs and units.
- **API clients** — conditional and mapped types for response shapes.
- **Type testing** — `expectTypeOf`, `tsd`, `ts-expect`.

### Core Concepts / Features

---

## 1. Generic Components

### Definitions

**Core Definition:** A generic component is a React function component parameterized by one or more type variables, allowing it to work with different data types while preserving type safety at each call site.

**Technical Definition:** A generic component declares a type parameter in its signature — `function List<T>(props: ListProps<T>)` — and TypeScript **infers `T`** from the props at the call site. Unlike `React.FC`, plain function declarations support generics. Type parameters can be constrained (`<T extends Identifiable>`) and defaulted (`<T = string>`). The inferred `T` flows through props, callbacks, and return values, so a `renderItem: (item: T) => ReactNode` callback receives the exact type of each item.

**Beginner-Friendly Explanation:** A generic component is one component that works with any type of data. `<List items={users} />` makes `T` a `User`; `<List items={products} />` makes `T` a `Product`. TypeScript figures it out automatically, so you get autocomplete and errors for the right type every time.

### Purposes

- To build components that work with any data type without sacrificing type safety.
- To link props together so that a callback receives the exact type of the item it processes.
- To avoid duplicating components for each data type.
- To constrain supported types with `extends` so invalid usage is a compile error.

### Syntax Rules and Structure

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => ReactNode;
  keyExtractor: (item: T) => string;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}
```

**Component Breakdown:**

| Part | Description |
|------|-------------|
| `<T>` | Type parameter declaration. |
| `ListProps<T>` | Props interface parameterized by `T`. |
| `function List<T>(...)` | Generic function component. |
| `T[]` | Array of `T` — items. |
| `(item: T) => ReactNode` | Renderer receives the inferred type. |

**Syntax Rules:**

1. Declare the type parameter on the function, not on `React.FC` (which cannot be generic).
2. Type the props interface using the same type parameter.
3. Constrain with `extends` when the component uses properties of `T`.
4. Provide defaults (`<T = string>`) only when a reasonable default exists.

**Constraints and Limitations:**

- `React.FC` does not support generic components — use plain function declarations.
- Inference may fail for deeply nested or restructured props; explicit `<Component<User> ...>` arguments help.
- Generic constraints that are too broad (`T extends object`) give vague errors; narrow them (`T extends { id: string }`).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Generic List Component

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => ReactNode;
  keyExtractor: (item: T) => string;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

// Usage — T is inferred as User
<List
  items={users}
  renderItem={(user) => user.name}   // user: User
  keyExtractor={(user) => user.id}
/>
```

**Expected Output:** `user` is typed as `User` inside `renderItem`; passing `items={[1, 2, 3]}` makes `user` a `number`, and `user.name` becomes a compile error.

#### Example 2: Constrained Generic Component

```tsx
interface Identifiable { id: string; }

interface TableProps<T extends Identifiable> {
  rows: T[];
  columns: { key: keyof T; header: string }[];
}

function Table<T extends Identifiable>({ rows, columns }: TableProps<T>) {
  return (
    <table>
      <thead>
        <tr>{columns.map((c) => <th key={String(c.key)}>{c.header}</th>)}</tr>
      </thead>
      <tbody>
        {rows.map((row) => (
          <tr key={row.id}>
            {columns.map((c) => <td key={String(c.key)}>{String(row[c.key])}</td>)}
          </tr>
        ))}
      </tbody>
    </table>
  );
}

// Usage — T must have an id
<Table
  rows={[{ id: '1', name: 'Alice', age: 30 }]}
  columns={[{ key: 'name', header: 'Name' }, { key: 'age', header: 'Age' }]}
/>
```

**Expected Output:** `columns` can only reference keys that exist on `T`; passing `{ key: 'email' }` when `T` has no `email` is a compile error.

### Real-World Cases

- **Data tables:** Generic tables with column definitions constrained to `keyof T`.
- **Select / combobox components:** Generic selects returning a typed `T` on change.
- **Form libraries:** React Hook Form and TanStack Form use generics for typed field values.

### References

- TypeScript Handbook — Generics — https://www.typescriptlang.org/docs/handbook/2/generics.html
- Steve Kinney — Generic Components — https://stevekinney.com/courses/react-typescript/generic-components
- React + TypeScript Cheatsheet — https://react-typescript-cheatsheet.netlify.app

---

## 2. Conditional Types

### Definitions

**Core Definition:** A conditional type is a type-level `if`-expression of the form `T extends U ? X : Y`, choosing `X` or `Y` based on whether `T` is assignable to `U`.

**Technical Definition:** Conditional types enable type-level branching and are the foundation of many built-in utilities (`Exclude`, `Extract`, `NonNullable`, `ReturnType`). Distribution over unions is **automatic and intentional**: when the checked type is a naked type parameter, the conditional distributes over each member of a union (`T extends U ? X : Y` applied to `A | B` yields `(A extends U ? X : Y) | (B extends U ? X : Y)`). The `infer` keyword extracts types from a position in `U`.

**Beginner-Friendly Explanation:** A conditional type says "if this type matches that type, use this other type; otherwise, use a third." It is TypeScript deciding types based on other types.

### Purposes

- To transform types based on their shape or membership.
- To extract or exclude union members (`Exclude`, `Extract`).
- To infer types from within other types (`infer`).
- To build utilities that adapt to the input type.

### Syntax Rules and Structure

```tsx
type IsString<T> = T extends string ? true : false;

type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type ElementTypeOf<T> = T extends React.ReactElement<infer P> ? P : never;
```

**Component Breakdown:**

| Part | Description |
|------|-------------|
| `T extends U` | Type-level check. |
| `? X : Y` | Branching. |
| `infer U` | Extract a type from `U`. |
| Distribution | Naked `T` distributes over unions automatically. |

**Syntax Rules:**

1. Wrap the checked type in a tuple (`[T] extends [U]`) to **prevent distribution** when you need an all-or-nothing check.
2. Use `infer` only inside the `extends` clause.
3. Nest conditionals carefully — deep nesting produces unreadable errors.
4. Prefer built-in utilities (`Exclude`, `Extract`, `ReturnType`) over reinventing them.

**Constraints and Limitations:**

- Conditional types can be expensive at compile time — heavy use slows editors.
- Error messages from deeply nested conditionals are hard to trace.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Extracting Props from a Component

```tsx
type ButtonProps = React.ComponentProps<typeof MyButton>;
// A common pattern — ComponentProps already handles this, but conditionals
// let you extract conditional shapes.

type NullableProps<T> = T extends null | undefined ? never : T;

interface Props {
  user: User | null;
  title: string;
}

type RequiredProps = {
  [K in keyof Props]: NullableProps<Props[K]>;
};
// { user: User; title: string } — null removed from user
```

**Expected Output:** `RequiredProps['user']` is `User` — the conditional strips `null`.

#### Example 2: Non-Distributive Check

```tsx
// Distributes: `A | B extends string ? ...` distributes over A and B
type Distributed<T> = T extends string ? 'text' : 'other';

// Non-distributive: wrap in tuple
type NonDistributed<T> = [T] extends [string] ? 'text' : 'other';

type A = Distributed<string | number>;    // 'text' | 'other'
type B = NonDistributed<string | number>; // 'other' (only if BOTH are strings)
```

**Expected Output:** `A` is `'text' | 'other'`; `B` is `'other'` because `string | number` is not fully assignable to `string`.

### Real-World Cases

- **Polymorphic components:** `T extends ElementType ? X : Y`.
- **Prop utilities:** `Omit`, `Pick` built on conditional and mapped types.
- **API clients:** `Response<T>` resolving to typed payloads.

### References

- TypeScript Handbook — Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html

---

## 3. Mapped Types

### Definitions

**Core Definition:** A mapped type generates a new object type by iterating over the keys of an existing type and optionally transforming the property types and modifiers.

**Technical Definition:** Mapped types use the syntax `{ [K in keyof T]: ... }` to produce a new type with the same keys as `T` but with transformed values. They support **key remapping** (`as` clause), modifier transformations (`+?`, `-?`, `+readonly`, `-readonly`), and value transformations via conditionals (`T[K] extends string ? ... : ...`). Mapped types are the mechanism behind `Partial<T>`, `Required<T>`, `Readonly<T>`, and `Record<K, V>`.

**Beginner-Friendly Explanation:** A mapped type takes all the keys of a type and transforms them — making them optional, required, readonly, or changing their values. `Partial<User>` and `Readonly<User>` are mapped types.

### Purposes

- To transform all properties of a type uniformly (optional, required, readonly).
- To build type utilities like `Partial`, `Pick`, `Omit`, `Record`.
- To generate new types with renamed keys (`as` clause).
- To model dynamic prop shapes in generic components.

### Syntax Rules and Structure

```tsx
// Make every property optional and nullable
type OptionalAndNullable<T> = {
  [K in keyof T]?: T[K] | null;
};

// Rename keys with template literal types
type EventHandlers<T> = {
  [K in keyof T as `on${Capitalize<string & K>}`]: (value: T[K]) => void;
};
```

**Component Breakdown:**

| Part | Description |
|------|-------------|
| `[K in keyof T]` | Iterate over all keys of `T`. |
| `T[K]` | The value type at key `K`. |
| `?` | Make the property optional. |
| `-?` | Remove optionality. |
| `as` | Remap the key. |
| `string & K` | Restrict to string keys before using string utilities. |

**Syntax Rules:**

1. Use `keyof T` to get the key union.
2. Use `as` to remap keys with template literal types.
3. Use modifier syntax (`+?`/`-?`, `+readonly`/`-readonly`) to change property modifiers.
4. Combine with conditional types for value transformations.

**Constraints and Limitations:**

- Mapped types are structural — they do not preserve class identity or method signatures automatically.
- Remapping keys can cause collisions — check for overlapping names.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Type-Safe Event Handler Map

```tsx
interface FormFields {
  name: string;
  email: string;
  age: number;
}

// Generate onChangeName, onChangeEmail, onChangeAge
type FormHandlers = {
  [K in keyof FormFields as `onChange${Capitalize<string & K>}`]: (
    value: FormFields[K]
  ) => void;
};

// Usage
const handlers: FormHandlers = {
  onChangeName: (v) => {},   // v: string
  onChangeEmail: (v) => {},  // v: string
  onChangeAge: (v) => {},    // v: number
};
```

**Expected Output:** `onChangeAge` receives a `number`; `onChangeName` receives a `string`. A typo like `onchangeName` is a compile error.

#### Example 2: Preserve Keys, Change Values

```tsx
type ToGetters<T> = {
  [K in keyof T]: () => T[K];
};

interface User { id: string; name: string; }
type UserGetters = ToGetters<User>;
// { id: () => string; name: () => string }
```

**Expected Output:** `UserGetters` contains `id` and `name` as functions returning `string`.

### Real-World Cases

- **Form libraries:** Generating typed handlers for each field.
- **Redux Toolkit slices:** Defining actions and reducers from a state shape.
- **API DTOs:** Deriving response shapes from request shapes.

### References

- TypeScript Handbook — Mapped Types — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html

---

## 4. Template Literal Types

### Definitions

**Core Definition:** Template literal types allow string literal types to be composed using template literal syntax, generating new string literal types from existing ones.

**Technical Definition:** Template literal types use backtick syntax with `${T}` interpolations where `T` is a string, number, or bigint literal type. They support the `Uppercase`, `Lowercase`, `Capitalize`, and `Uncapitalize` intrinsic types for case transformation. Combined with mapped types, they enable strongly typed string formats, event names, CSS property names, and API routes.

**Beginner-Friendly Explanation:** A template literal type builds strings at the type level — like `` `on${Capitalize<'click'>}` `` producing `'onClick'`. It lets TypeScript understand string patterns.

### Purposes

- To generate typed event names, CSS properties, or route patterns.
- To enforce string formats (e.g., hex colors, CSS custom properties).
- To build strongly typed APIs for dynamic keys.
- To derive prop names from data keys.

### Syntax Rules and Structure

```tsx
type EventName = 'click' | 'focus' | 'blur';
type HandlerName = `on${Capitalize<EventName>}`; // 'onClick' | 'onFocus' | 'onBlur'

type CSSVar = `--${string}`;
const varName: CSSVar = '--primary-color';
```

**Component Breakdown:**

| Part | Description |
|------|-------------|
| `` `...` `` | Template literal syntax. |
| `${T}` | Type interpolation. |
| `Capitalize<T>` / `Uppercase<T>` | Case transformation. |
| `string & K` | Restrict to string keys. |

**Syntax Rules:**

1. Interpolations must be string, number, or bigint literal types.
2. Template literal types produce literal unions when applied to unions.
3. Use with mapped types to rename keys or generate families of types.
4. Use `string & K` when the key type may include symbols.

**Constraints and Limitations:**

- Extremely large unions of template literal types can slow compilation.
- The compiler cannot solve arbitrary string equations — patterns must be decomposable.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Typed Event Handlers

```tsx
interface User {
  id: string;
  name: string;
}

type UserChangeHandlers = {
  [K in keyof User as `on${Capitalize<K & string>}Change`]: (value: User[K]) => void;
};

// { onIdChange: (v: string) => void; onNameChange: (v: string) => void }
```

**Expected Output:** `onNameChange` is typed to receive `string`; typos like `onameChange` are compile errors.

#### Example 2: CSS Custom Property Type

```tsx
type CSSVar = `--${string}`;

function setCSSVar(name: CSSVar, value: string) {
  document.documentElement.style.setProperty(name, value);
}

setCSSVar('--primary-color', '#ff0000'); // ✅
setCSSVar('primary-color', '#ff0000');   // ❌ compile error — missing '--'
```

**Expected Output:** Only strings beginning with `--` are accepted.

### Real-World Cases

- **Design tokens:** Typed CSS variable names.
- **Typed event systems:** Generating `on${Event}` handler names.
- **Query builders:** Typed SQL/GraphQL column references.

### References

- TypeScript Handbook — Template Literal Types — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html

---

## 5. Type Inference

### Definitions

**Core Definition:** Type inference is TypeScript's ability to automatically determine a type without an explicit annotation, based on how a value is used.

**Technical Definition:** Inference flows from initializers (`const x = 5` → `number`), function arguments (generic inference), return expressions, contextual positions (expected types in callbacks and JSX attributes), and control flow (narrowing). Advanced inference scenarios include **inference in generic functions** (TypeScript infers `T` from arguments), **contextual typing of callbacks** (e.g., `.map(item => ...)` — the parameter type comes from the array), and **`infer` inside conditional types** to extract types from structure.

**Beginner-Friendly Explanation:** TypeScript is very good at guessing types from context. `const users = []; users.push('a')` — inference makes `users` a `string[]`. In React, event handlers inherit the event type from the JSX attribute.

### Purposes

- To reduce boilerplate — annotate only where needed.
- To propagate the correct types through generic calls.
- To support contextual typing of inline callbacks.
- To enable precise type extraction with `infer`.

### Syntax Rules and Structure

```tsx
// Inference from initializer
const count = 0;                 // number

// Inference from generic call
function identity<T>(value: T): T { return value; }
const x = identity('hello');     // string

// Inference in conditional types
type UnwrapArray<T> = T extends (infer U)[] ? U : T;
type Item = UnwrapArray<User[]>; // User

// Contextual typing in JSX
<button onClick={(e) => e.currentTarget.disabled = true} />
// e is React.MouseEvent<HTMLButtonElement>
```

**Constraints and Limitations:**

- `useState(null)` infers `null` — provide a generic.
- Widening (`const` vs `let`) affects literal types — use `as const` for literal inference.
- Sometimes inference chooses the union of possible types rather than the specific one intended.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Generic Inference

```tsx
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
  const result = {} as Pick<T, K>;
  for (const key of keys) result[key] = obj[key];
  return result;
}

const user = { id: '1', name: 'Alice', age: 30 };
const picked = pick(user, ['id', 'name']);
// picked: { id: string; name: string }
```

**Expected Output:** `picked` is inferred as `Pick<User, 'id' | 'name'>` — no explicit type annotations needed.

### Real-World Cases

- **API clients:** `fetchJson<T>()` infers `T` from the assignment target.
- **Form hooks:** TanStack Form infers field names from the schema.
- **Type extraction:** `ComponentProps<typeof C>` extracts props.

### References

- TypeScript Handbook — Type Inference — https://www.typescriptlang.org/docs/handbook/type-inference.html

---

## 6. Type Composition

### Definitions

**Core Definition:** Type composition is building complex types from smaller types using intersections (`&`), unions (`|`), conditional types, and mapped types.

**Technical Definition:** Type composition combines types while preserving their constraints: **intersections** merge object shapes; **unions** describe alternatives; **mapped types** transform whole shapes; **conditional types** branch on structure; **template literal types** compose strings. Composition is the alternative to deep inheritance and duplication — each piece expresses one concern and composes into the final type.

**Beginner-Friendly Explanation:** Type composition is building big types out of small ones — like combining "User fields" and "Admin fields" into an "AdminUser" type.

### Purposes

- To express complex shapes without duplicating definitions.
- To separate concerns (BaseProps, InteractiveProps, StylingProps) and combine them.
- To make types easier to refactor (change one piece, propagate everywhere).
- To model domain hierarchies and multi-inheritance-like structures.

### Syntax Rules and Structure

```tsx
interface BaseProps { className?: string; children?: ReactNode; }
interface InteractiveProps { onClick?: () => void; disabled?: boolean; }
interface VisualProps { variant?: 'primary' | 'secondary'; size?: 'sm' | 'md'; }

type ButtonProps = BaseProps & InteractiveProps & VisualProps;
```

**Constraints and Limitations:**

- Intersections with conflicting property types produce `never` for that property.
- For object extension, `interface extends` is often clearer than intersection aliases.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Composing a Button

```tsx
interface BaseProps { className?: string; }
interface DisableableProps { disabled?: boolean; }
interface ClickableProps { onClick: () => void; }

type ButtonProps = BaseProps & DisableableProps & ClickableProps & {
  label: string;
};
```

**Expected Output:** `ButtonProps` requires `onClick` and `label`, allows `disabled` and `className`. Each concern is expressed separately and composable across components.

### Real-World Cases

- **Design system primitives:** Base × interactive × visual props.
- **API schemas:** Combining common pagination/filter props with specific resource props.
- **HOC patterns:** Composing wrapper and wrapped props.

### References

- TypeScript Handbook — Intersection Types — https://www.typescriptlang.org/docs/handbook/2/objects.html#intersection-types

---

## 7. Type-Level Programming

### Definitions

**Core Definition:** Type-level programming is the practice of writing TypeScript types that compute other types — essentially programs that run at compile time and produce types.

**Technical Definition:** Type-level programming uses conditional types, mapped types, recursion, `infer`, and template literal types to transform types. Common examples include string manipulation at the type level (`Split`, `Join`), arithmetic (`Add`, `Multiply`), sorting, and validation (parse a URL pattern type and extract params). Type-level programs run entirely at compile time and produce zero runtime cost — but they can slow compilation if overused.

**Beginner-Friendly Explanation:** Type-level programming means writing types that act like little programs — for example, a type that splits a string into its characters, or one that computes the type of a URL's params from its pattern.

### Purposes

- To express complex invariants that simple types cannot capture.
- To derive types from data patterns (routes, events, tokens).
- To validate API contracts at compile time.
- To extract useful information from structures (`infer`).

### Syntax Rules and Structure

**Example — string splitting:**

```tsx
type Split<S extends string, D extends string> =
  S extends `${infer Head}${D}${infer Tail}`
    ? [Head, ...Split<Tail, D>]
    : [S];

type Parts = Split<'a-b-c', '-'>; // ['a', 'b', 'c']
```

**Example — route params:**

```tsx
type ExtractParams<Path extends string> =
  Path extends `${string}:${infer Param}/${infer Rest}`
    ? { [K in Param | keyof ExtractParams<`/${Rest}`>]: string }
    : Path extends `${string}:${infer Param}`
    ? { [K in Param]: string }
    : {};
```

**Constraints and Limitations:**

- Type recursion depth is limited (~50 levels by default) — deep recursion fails to compile.
- Compilation time grows with type complexity — heavy type-level code slows editors and CI.
- Error messages from type-level code are extremely hard to debug.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Extracting Route Params

```tsx
type RouteParams<Path extends string> =
  Path extends `${string}/:${infer Param}/${infer Rest}`
    ? { [K in Param | keyof RouteParams<`/${Rest}`>]: string }
    : Path extends `${string}/:${infer Param}`
      ? { [K in Param]: string }
      : {};

type Params = RouteParams<'/users/:userId/posts/:postId'>;
// { userId: string; postId: string }
```

**Expected Output:** `Params` has exactly `userId` and `postId` as `string` — no other keys.

### Real-World Cases

- **Typed routers:** Extracting params from route paths at compile time.
- **GraphQL clients:** Deriving selection sets from queries.
- **ORM schemas:** Inferring model types from table definitions.

### References

- TypeScript Handbook — Conditional Types (recursive patterns) — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- Type Challenges — https://github.com/type-challenges/type-challenges

---

## 8. Branded Types

### Definitions

**Core Definition:** A branded type (also called a **tagged** or **nominal** type) attaches a phantom marker to a primitive type to enforce that structurally identical types are treated as distinct.

**Technical Definition:** TypeScript is structural — `type UserId = string` and `type OrderId = string` are interchangeable. Branding uses an intersection with a "brand" property that never exists at runtime: `type UserId = string & { readonly __brand: 'UserId' }`. Values can only be produced by a **constructor function** that casts (`as UserId`), typically after validation. Brands can be **opaque** (the brand is not exposed) or **transparent** (the brand is documented). This gives nominal typing on top of a structural system.

**Beginner-Friendly Explanation:** Branded types make TypeScript treat two strings as different — a `UserId` is not an `OrderId`, even though both are strings. This prevents mixing up IDs across domains, a common source of bugs.

### Purposes

- To prevent mixing up values that share an underlying type (IDs, tokens, units).
- To enforce validation — a branded value can only be created via its constructor.
- To model domain concepts precisely (USD vs. EUR, Celsius vs. Fahrenheit).
- To catch unit-mismatch bugs at compile time.

### Syntax Rules and Structure

```tsx
// Brand definition
type UserId = string & { readonly __brand: 'UserId' };
type OrderId = string & { readonly __brand: 'OrderId' };

// Constructor
function asUserId(id: string): UserId {
  return id as UserId;
}

function asOrderId(id: string): OrderId {
  return id as OrderId;
}

// Functions
function getUser(id: UserId) { /* ... */ }

const u = asUserId('u1');
const o = asOrderId('o1');

getUser(u); // ✅
getUser(o); // ❌ compile error — OrderId is not UserId
```

**Component Breakdown:**

| Part | Description |
|------|-------------|
| `string & { readonly __brand: 'UserId' }` | Structural intersection with a unique brand. |
| `as UserId` | The only way to create a branded value. |
| Constructor function | Encapsulates creation and validation. |

**Constraints and Limitations:**

- Brands do not exist at runtime — they are purely compile-time.
- Every brand needs a constructor — uncontrolled `as` casts defeat the purpose.
- For opaque brands, keep the brand key private and expose only constructors.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Validated Branded IDs

```tsx
type UserId = string & { readonly __brand: unique symbol };

function asUserId(id: string): UserId {
  if (!id.startsWith('user_')) throw new Error('Invalid UserId');
  return id as UserId;
}

function findUser(id: UserId): User { /* ... */ }

const id = asUserId('user_123'); // ✅ after validation
findUser(id);

// findUser('user_123'); // ❌ plain string is not UserId
```

**Expected Output:** `findUser` accepts only `UserId`, and only values created via `asUserId` qualify.

### Real-World Cases

- **Multi-tenant apps:** `TenantId` and `UserId` cannot be swapped.
- **Financial systems:** `USD` and `EUR` amounts are distinct types.
- **Healthcare:** `PatientId` and `ProviderId` remain distinct.

### References

- TypeScript — Branded Types Pattern — https://www.typescriptlang.org/docs/handbook/2/objects.html
- TypeScript Playbook — Branded Types — https://www.totaltypescript.com/tips/

---

## 9. Opaque Types

### Definitions

**Core Definition:** An opaque type is a branded type whose brand marker is **not visible** to consumers — they can only create and use values via the module's exported constructors and functions.

**Technical Definition:** An opaque type is created in a module by defining a branded type with a **private brand** (often a unique symbol) and exporting only the constructor functions and accessors. Consumers can hold opaque values and pass them back to the module but cannot fabricate them or access their internal structure. This is the TypeScript equivalent of **abstract data types** in functional programming.

**Beginner-Friendly Explanation:** An opaque type is a branded type where the brand is hidden — you can only get one by asking the module that defines it. It is a way of saying "this value is valid because the module validated it."

### Purposes

- To hide the internal representation of a domain type.
- To enforce that values are only created through validated constructors.
- To allow the implementation to change without breaking consumers.
- To model abstract data types (money, dates, paths) safely.

### Syntax Rules and Structure

```tsx
// money.ts — module defining opaque Money type
declare const MoneyBrand: unique symbol;

export type Money = { readonly amount: number; readonly [MoneyBrand]: 'Money' };

export function createMoney(amount: number): Money {
  if (!Number.isFinite(amount)) throw new Error('Invalid amount');
  return { amount, [MoneyBrand]: 'Money' } as Money;
}

export function addMoney(a: Money, b: Money): Money {
  return createMoney(a.amount + b.amount);
}
```

**Constraints and Limitations:**

- The brand must be kept private — leaking the brand symbol breaks the opacity guarantee.
- Opaque types have a runtime representation unless you use brand-only intersections on primitives.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Opaque Money Type

```tsx
// Opaque Money — only constructible via createMoney
declare const MoneyBrand: unique symbol;
export type Money = { readonly amount: number; readonly [MoneyBrand]: true };

export function createMoney(amount: number): Money {
  return { amount, [MoneyBrand]: true } as Money;
}

// consumer
import { createMoney, Money } from './money';
const price = createMoney(100);   // ✅
// const fake: Money = { amount: 100 }; // ❌ missing brand
```

**Expected Output:** `Money` cannot be fabricated by consumers — only `createMoney` produces a valid `Money`.

### Real-World Cases

- **Financial libraries:** Opaque `Currency`, `Amount`, `Rate` types.
- **Date libraries:** Opaque `DateOnly`, `DateTime` types.
- **Path libraries:** Opaque `AbsolutePath`, `RelativePath` types.

### References

- TypeScript — Opaque Types Pattern — https://www.totaltypescript.com/

---

## 10. Discriminated Unions

### Definitions

**Core Definition:** A discriminated union is a union of object types where each member has a common **literal discriminant property** (usually `type` or `kind`) that TypeScript can use to narrow to the correct member.

**Technical Definition:** In a discriminated union, each member shares a property whose type is a unique literal (e.g., `status: 'success'`). TypeScript's control-flow analysis narrows the union based on comparisons on this discriminant property. This enables type-safe access to member-specific properties inside `if` and `switch` blocks and enables **exhaustive checking** with a `never` guard.

**Beginner-Friendly Explanation:** A discriminated union is a set of object shapes that share a `type` field. TypeScript uses that field to figure out which shape you have, so you can safely access the properties unique to that shape.

### Purposes

- To model states (loading/success/error) or actions with type safety.
- To enable exhaustive switch statements with `never` checks.
- To prevent accessing properties that do not exist for the current state.
- To make invalid states unrepresentable.

### Syntax Rules and Structure

```tsx
type State =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: User[] }
  | { status: 'error'; error: Error };
```

**Constraints and Limitations:**

- The discriminant must be a **literal type** (not `string` or `number`).
- All members must declare the discriminant (unless optionality is intended).
- Use `never` in `default` to guarantee exhaustiveness.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Render by State

```tsx
function render(state: State) {
  switch (state.status) {
    case 'idle':
      return <p>Ready</p>;
    case 'loading':
      return <p>Loading...</p>;
    case 'success':
      return <ul>{state.data.map((u) => <li key={u.id}>{u.name}</li>)}</ul>;
    case 'error':
      return <p role="alert">{state.error.message}</p>;
  }
}
```

**Expected Output:** `state.data` is only accessible in the `'success'` case; accessing it in `'loading'` is a compile error.

### Real-World Cases

- **Redux actions:** Every action has a unique literal `type`.
- **UI state machines:** Loading, success, error, empty.
- **API responses:** Success and error shapes.

### References

- TypeScript Handbook — Discriminated Unions — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- Steve Kinney — Discriminated Unions — https://stevekinney.com/courses/react-typescript/typescript-discriminated-unions

---

## 11. Exhaustive Checks

### Definitions

**Core Definition:** An exhaustive check is a compile-time guarantee that every member of a union has been handled in a `switch` or `if` chain.

**Technical Definition:** Exhaustiveness uses the **`never` type** as a compile-time assert. In the `default` branch of a `switch` over a discriminated union, `const _exhaustive: never = action;` produces a compile error if any member is unhandled, because the remaining type is not `never`. This catches regressions when new union members are added.

**Beginner-Friendly Explanation:** An exhaustive check is a line of code that ensures you handle every case. If someone adds a new state later, the compiler tells you to handle it too.

### Purposes

- To guarantee that every union member is handled.
- To catch missing `case` branches when new members are added.
- To make refactoring safe — adding a new state forces handling everywhere it matters.

### Syntax Rules and Structure

```tsx
function assertNever(value: never): never {
  throw new Error(`Unhandled case: ${JSON.stringify(value)}`);
}

function handle(action: Action) {
  switch (action.type) {
    case 'increment': return;
    case 'decrement': return;
    default: assertNever(action); // compile error if any case is missing
  }
}
```

**Constraints and Limitations:**

- The `default` branch must be reachable at runtime — add a runtime throw for safety.
- Exhaustiveness only works when the union is closed (not extended elsewhere).

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: `assertNever` Helper

```tsx
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; side: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle': return Math.PI * shape.radius ** 2;
    case 'square': return shape.side ** 2;
    default: return assertNever(shape);
  }
}
```

**Expected Output:** Adding `{ kind: 'triangle'; ... }` to `Shape` produces a compile error at `assertNever(shape)` — the new case must be handled.

### Real-World Cases

- **Reducers:** Ensuring every action is handled.
- **Renderers:** Ensuring every UI state renders something.
- **API parsers:** Ensuring every response variant is processed.

### References

- TypeScript Handbook — Exhaustiveness Checking — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking

---

## 12. Type Testing

### Definitions

**Core Definition:** Type testing is the practice of writing tests that verify **types behave as intended** — that a utility type resolves to the correct shape, that a component rejects invalid props, and that changes to types do not silently break.

**Technical Definition:** Type testing frameworks include **`vitest`'s `expectTypeOf`** (inline assertions in test files), **`tsd`** (standalone type tests in `.test-d.ts` files), and **`ts-expect`** (assertion-based, framework-agnostic). Type tests validate: assignability, inference results, error cases (using `// @ts-expect-error`), branded type invariants, and utility type outputs. They run at compile time (`tsc --noEmit`), so any break in type behavior fails CI.

**Beginner-Friendly Explanation:** Type testing is writing tests that check the **types** — not runtime behavior. If someone changes a utility type and it silently starts accepting the wrong values, the type test catches it.

### Purposes

- To verify that utility and conditional types resolve correctly.
- To catch regressions in generic inference.
- To validate branded types cannot be mixed.
- To document type contracts as executable assertions.

### Syntax Rules and Structure

```tsx
// Vitest — expectTypeOf
import { expectTypeOf, test } from 'vitest';

test('Pick resolves to the correct shape', () => {
  type Picked = Pick<User, 'id' | 'name'>;
  expectTypeOf<Picked>().toEqualTypeOf<{ id: string; name: string }>();
});

// tsc-only errors — assert that invalid usage is a compile error
// @ts-expect-error — UserId is required
getUser('not-a-userid');
```

**Constraints and Limitations:**

- `// @ts-expect-error` fails if the line does **not** produce an error — a valuable guarantee.
- Type tests run through `tsc --noEmit` — they must be included in the tsconfig.
- `expectTypeOf` assertions are compile-time only; they do not execute at runtime.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Testing a Conditional Type

```tsx
// typeUtils.test.ts
import { expectTypeOf, test } from 'vitest';

type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

test('UnwrapPromise extracts the resolved type', () => {
  expectTypeOf<UnwrapPromise<Promise<string>>>().toEqualTypeOf<string>();
  expectTypeOf<UnwrapPromise<number>>().toEqualTypeOf<number>();
});

test('UnwrapPromise does not accept non-promises as promises', () => {
  // @ts-expect-error — UnwrapPromise<number> is number, not Promise
  const p: UnwrapPromise<number> = Promise.resolve(1);
});
```

**Expected Output:** All type assertions pass. If `UnwrapPromise` were changed to return `unknown`, the test fails at compile time.

### Real-World Cases

- **Utility type libraries:** Every utility has type tests.
- **Generic components:** Type tests verify inference.
- **Branded types:** Type tests assert that `UserId` cannot be assigned to `OrderId`.

### References

- Vitest — `expectTypeOf` — https://vitest.dev/api/expect-typeof.html
- tsd — https://github.com/SamVerschueren/tsd
- ts-expect — https://github.com/TypeStrong/ts-expect

---

## 13. Advanced TypeScript Best Practices

### Definitions

**Core Definition:** Advanced TypeScript best practices are the guidelines that keep complex type-level code safe, readable, performant, and maintainable.

**Technical Definition:** Best practices include: **prefer inference** to explicit annotations unless clarity requires otherwise; **constrain generics** narrowly (`T extends { id: string }` rather than `T extends object`); **avoid over-clever type-level programs** — readability beats cleverness; **document complex types** with JSDoc; **test types** with `expectTypeOf` and `tsd`; **prefer built-in utilities** (`Pick`, `Omit`, `ComponentProps`) over reinventing them; **avoid recursion depth issues** by simplifying or flattening; **use branded types for domain distinctions**; **use discriminated unions** for state; and **enable `strict` plus `noUncheckedIndexedAccess`** and `exactOptionalPropertyTypes`.

**Beginner-Friendly Explanation:** Advanced types are powerful, but complex types are hard to read and slow to compile. Best practices keep the type system working for you instead of against you — use inference when possible, keep generics constrained, test your types, and do not over-engineer.

### Purposes

- To keep complex type code maintainable over time.
- To preserve fast editor and CI feedback.
- To catch type regressions early.
- To balance flexibility with strictness.

### Best Practices Checklist

| Practice | Description |
|----------|-------------|
| **Prefer inference** | Annotate where inference is wrong or unclear. |
| **Narrow generics** | Constrain to what the component actually uses. |
| **Avoid cleverness** | Readable types beat brittle type-level programs. |
| **Test types** | `expectTypeOf` / `tsd` for utility and conditional types. |
| **Prefer built-ins** | `Pick`, `Omit`, `Record`, `ComponentProps`, `Parameters`, `ReturnType`. |
| **Document complex types** | JSDoc with examples. |
| **Watch compile cost** | Overly recursive types slow editors. |
| **Brand domain types** | Prevent ID/unit mix-ups. |
| **Discriminated unions** | Model states; enforce exhaustiveness. |
| **Strict tsconfig** | `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`. |

**Constraints and Limitations:**

- Strict flags can feel noisy in legacy code — introduce them incrementally.
- Advanced types are a maintenance burden — reserve them for cases where the safety clearly outweighs the cost.

### Multiple Annotated Complete Step by Step Code Examples

#### Example 1: Documented Advanced Type

```tsx
/**
 * Extracts the props of a React component, omitting `className` and `style`.
 *
 * @example
 * type MyButtonProps = CleanProps<typeof MyButton>;
 * // { label: string; onClick: () => void }
 */
type CleanProps<C extends React.ElementType> =
  Omit<React.ComponentPropsWithoutRef<C>, 'className' | 'style'>;
```

**Why this follows best practices:** JSDoc explains intent with an example; the type uses built-in utilities (`Omit`, `ComponentPropsWithoutRef`) rather than reinventing them; the generic is constrained to `React.ElementType`.

### Real-World Cases

- **Design systems:** Complex polymorphic components with documented props types.
- **Utility libraries:** Type-level utilities with test coverage.
- **Domain modeling:** Branded types and discriminated unions across the codebase.

### References

- TypeScript Handbook — https://www.typescriptlang.org/docs/handbook/
- Type Challenges — https://github.com/type-challenges/type-challenges
- React + TypeScript Cheatsheet — https://react-typescript-cheatsheet.netlify.app

---

## References

- TypeScript Handbook — Generics — https://www.typescriptlang.org/docs/handbook/2/generics.html
- TypeScript Handbook — Conditional Types — https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
- TypeScript Handbook — Mapped Types — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- TypeScript Handbook — Template Literal Types — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- TypeScript Handbook — Type Inference — https://www.typescriptlang.org/docs/handbook/type-inference.html
- TypeScript Handbook — Intersection Types — https://www.typescriptlang.org/docs/handbook/2/objects.html#intersection-types
- TypeScript Handbook — Discriminated Unions — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions
- TypeScript Handbook — Exhaustiveness Checking — https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking
- TypeScript — tsconfig.json Reference — https://www.typescriptlang.org/tsconfig/
- Steve Kinney — Generic Components — https://stevekinney.com/courses/react-typescript/generic-components
- Steve Kinney — Discriminated Unions — https://stevekinney.com/courses/react-typescript/typescript-discriminated-unions
- Steve Kinney — Polymorphic Components and the `as` Prop — https://stevekinney.com/courses/react-typescript/polymorphic-components-and-as-prop
- React + TypeScript Cheatsheet — https://react-typescript-cheatsheet.netlify.app
- Type Challenges — https://github.com/type-challenges/type-challenges
- Vitest — `expectTypeOf` — https://vitest.dev/api/expect-typeof.html
- tsd — https://github.com/SamVerschueren/tsd
- ts-expect — https://github.com/TypeStrong/ts-expect
- Total TypeScript — Branded Types — https://www.totaltypescript.com/