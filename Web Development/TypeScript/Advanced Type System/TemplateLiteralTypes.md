# Template Literal Types in TypeScript — Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition (Academic).** A **template literal type** is a type-level construct that mirrors JavaScript's template literal syntax—backtick-delimited strings with `${...}` interpolation positions—but operates entirely in the **type space**. Template literal types **build on string literal types** and can **expand into many strings via unions**, producing new string literal types by concatenating and transforming existing literal types.

**Technical Definition.** Template literal types were introduced in **TypeScript 4.1**. They have the same syntax as JavaScript template literals but are used in **type positions**. When interpolation positions contain **union types**, the resulting type is the **set of every possible string literal** that could be represented by each union member—template literals **distribute over unions**, producing a **Cartesian product** of the interpolated unions. TypeScript provides four **intrinsic string manipulation types**—`Uppercase<T>`, `Lowercase<T>`, `Capitalize<T>`, and `Uncapitalize<T>`—that transform string literal types. When combined with **conditional types** and the **`infer` keyword**, template literal types enable **pattern matching, extraction, and recursive string parsing** at the type level.

**Beginner-Friendly Explanation.** A template literal type is **a string pattern that lives in the type system**. You write it exactly like a JavaScript template literal—`` `Hello ${Name}` ``—but TypeScript treats it as a type. If `Name` is `"world"`, the type is `"Hello world"`. If `Name` is `"Alice" | "Bob"`, the type expands to `"Hello Alice" | "Hello Bob"`. You can use it to **type-check strings that follow a pattern**, such as event handler names (`onClick`), CSS values (`100px`), API routes (`/users/:id`), and environment variable keys (`API_USER_URL`). The compiler enforces the pattern at every call site, catching typos before runtime.

---

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Type-level string construction** | Builds new string literal types by concatenating and interpolating existing types. |
| **Union distribution** | Interpolated unions are **cross-multiplied**, producing every combination. |
| **Intrinsic string utilities** | `Uppercase`, `Lowercase`, `Capitalize`, and `Uncapitalize` transform literal strings. |
| **Pattern matching with `infer`** | Conditional types can decompose structured strings into named pieces. |
| **Composability** | Works with generics, mapped types, conditional types, and key remapping (`as`). |
| **Combinatorial sensitivity** | Large unions multiply into huge literal sets, causing compiler slowdowns. |
| **Erasure** | All template literal types are compile-time only and emit no JavaScript. |

---

### Prerequisites

- **TypeScript basics**: type aliases, union types, and literal types.
- **String literal types**: exact string values as types.
- **Generics fundamentals**: type parameters and constraints.
- **Conditional types** and the **`infer` keyword** for pattern extraction.
- **Mapped types** for key remapping with `as`.
- **A TypeScript environment** (`tsc` or an editor with TypeScript support).

---

### Related Programming Areas

- **Mapped types and key remapping** — template literal types rename properties via `as` clauses.
- **Conditional types and `infer`** — decompose structured strings into named parts.
- **String literal unions** — the building blocks of template literal types.
- **Type-level string algorithms** — recursive splitting, joining, and replacing.
- **Compiler performance** — combinatorial explosion and recursion limits are the primary constraints.

---

## Core Concepts / Features

---

## 1. Template Literal Types

### Definitions

**Core Definition.** A **template literal type** is a type expression with backtick syntax and `${...}` interpolation positions that produces a **new string literal type** by substituting the interpolated types.

**Technical Definition.** Template literal types use the same syntax as JavaScript template literals but operate in type positions. With concrete literal types, they concatenate the contents into a single literal type. With union types in interpolation positions, they **cross-multiply** the unions, producing the set of all possible combinations. They may be combined with generic type parameters, conditional types, and intrinsic string utilities.

**Beginner-Friendly Explanation.** A template literal type is **a fill-in-the-blank string type**. `` `Hello ${T}` `` is a pattern: `T` gets filled in with whatever literal type is supplied. If `T` is a union, you get every possible filled-in string.

### Purposes

- **To model functions and APIs that expect a set of specific string patterns**, such as event names, CSS values, and API routes.
- **To derive new string literal types** from existing unions without manual enumeration.
- **To enforce string format constraints** at compile time, catching typos and invalid values before runtime.
- **To enable type-safe string manipulation** through intrinsic utilities and pattern matching.
- **To build generative types** that stay synchronized with source unions.

### Syntax Rules and Structure

**General syntax:**

```
`prefix${Type}suffix`
```

**Component breakdown:**

- `` ` `` — backtick delimiters (same as JavaScript template literals).
- `prefix`, `suffix` — literal string text.
- `${Type}` — an interpolated **type** position.
- The result is a string literal type (or union of literals).

**Syntax rules.**

- Template literal types use **backticks**, not quotes.
- Interpolated positions accept **types**: literal types, unions, `string`, `number`, `bigint`, `boolean`, `null`, `undefined`, and other template literal types.
- Unions in interpolation positions are **distributed** (cross-multiplied).
- Intrinsic string utilities may be applied to interpolated types.

**Constraints and limitations.**

- Template literal types cannot generate **infinite** string sets; use `string` for unbounded patterns.
- Large unions cause **combinatorial explosion** (see best practices).
- Recursive template literal types can hit compiler limits (~50 levels of instantiation depth).

### Annotated Complete Step by Step Code Examples

#### Example 1: Basic concatenation

```typescript
// example_template_basic.ts
// Step 1: Define a literal type.
type World = "world";

// Step 2: Interpolate it into a template literal type.
type Greeting = `hello ${World}`; // "hello world"

// Step 3: Use the resulting type.
const greeting: Greeting = "hello world";
console.log(greeting); // Expected: hello world
```

**Setup.** Compile with `npx tsc example_template_basic.ts --strict`.

**Expected Output:**
```
hello world
```

**Why this output occurs.** The template literal type concatenates `"hello "` with the literal type `"world"`, producing exactly `"hello world"`.

---

#### Example 2: Union cross-multiplication

```typescript
// example_template_union.ts
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

// Each union member is cross-multiplied with the "_id" suffix.
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
// "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"

const id: AllLocaleIDs = "footer_title_id";
console.log(id); // Expected: footer_title_id
```

**Expected Output:**
```
footer_title_id
```

**Why this output occurs.** The union in the interpolation position expands to every possible combination with the `"_id"` suffix, producing four literal types.

---

### Real-World Cases

- **CSS-in-JS values**: `` type CSSValue = `${number}px` | `${number}em` | `${number}%` ``.
- **Environment variable keys**: `` `API_${Uppercase<Endpoint>}_URL` ``.
- **Tailwind-style class combinations**: `` `btn-${Color}-${Size}` ``.

---

## 2. `` `Hello ${T}` `` — Interpolation Syntax

### Definitions

**Core Definition.** The interpolation syntax `` `${T}` `` inserts a type `T` into a template literal type at a specific position, producing a new string literal type or union.

**Technical Definition.** Each `${...}` position is a **type substitution site**. The interpolated type may be a literal, union, intrinsic string transformation, or another template literal type. When multiple interpolation positions exist, the result is the **Cartesian product** of all interpolated unions.

**Beginner-Friendly Explanation.** `` `${T}` `` is **the blank you fill in**. You put a type inside the braces, and TypeScript substitutes every possible literal value of that type into the string pattern.

### Purposes

- **To compose string types** from reusable literal unions.
- **To insert computed string transformations** (`Uppercase`, `Capitalize`) into patterns.
- **To build multi-dimensional patterns** by interpolating several unions.
- **To parameterize string patterns** with generic type variables.

### Syntax Rules and Structure

```typescript
type Pattern<T extends string> = `prefix-${T}-suffix`;
```

**Multiple interpolations:**

```typescript
type Combo<A extends string, B extends string> = `${A}-${B}`;
```

**With intrinsic utilities:**

```typescript
type Handler<E extends string> = `on${Capitalize<E>}`;
```

**Component breakdown:**

- `${T}` — interpolation position.
- `T` — a type parameter or literal union.
- `prefix-` / `-suffix` — literal text around the interpolation.

**Syntax rules.**

- Interpolated types must be assignable to `string | number | bigint | boolean | null | undefined`.
- Multiple interpolation positions multiply their unions.
- Intrinsic utilities transform literal types before insertion.

**Constraints and limitations.**

- `number` and `bigint` interpolations produce string representations of numeric literals, which can be large.
- Symbol types cannot be interpolated directly.

### Annotated Complete Step by Step Code Examples

#### Example 1: Event handler names with `Capitalize`

```typescript
// example_interpolation_capitalize.ts
type DOMEvents = "click" | "focus" | "blur" | "change";

type EventHandlerName = `on${Capitalize<DOMEvents>}`;
// "onClick" | "onFocus" | "onBlur" | "onChange"

const handler: EventHandlerName = "onClick";
console.log(handler); // Expected: onClick

// const bad: EventHandlerName = "onHover"; // ❌ Error
```

**Expected Output:**
```
onClick
```

**Why this output occurs.** `Capitalize<"click">` produces `"Click"`, and the template literal prepends `"on"`, yielding `"onClick"`. The union distributes over all four event names.

---

#### Example 2: Multi-dimensional class names

```typescript
// example_interpolation_multi.ts
type Size = "sm" | "md" | "lg";
type Color = "primary" | "secondary" | "danger";

type ButtonClass = `btn-${Color}-${Size}`;
// 3 × 3 = 9 members: "btn-primary-sm", ..., "btn-danger-lg"

const cls: ButtonClass = "btn-primary-md";
console.log(cls); // Expected: btn-primary-md

// const bad: ButtonClass = "btn-primary-xl"; // ❌ Error
```

**Expected Output:**
```
btn-primary-md
```

**Why this output occurs.** Each union is cross-multiplied, producing every combination of color and size.

---

### Real-World Cases

- **React event handlers**: `` `on${Capitalize<ElementEvent>}` ``.
- **Button variants**: `` `btn-${Color}-${Size}` ``.
- **API environment keys**: `` `API_${Uppercase<Endpoint>}_URL` ``.

---

## 3. Template Literal Type Declaration

### Definitions

**Core Definition.** A **template literal type declaration** is a type alias that defines a template literal type, typically with generic parameters, so the pattern can be reused.

**Technical Definition.** Declarations use `type Name<T> = \`...${T}...\``. They may be exported, constrained (`T extends string`), and composed with conditional types and intrinsic utilities. Declarations are **erased** and produce no runtime code.

**Beginner-Friendly Explanation.** A template literal type declaration is **giving a name to a string pattern** so you can reuse it with different inputs.

### Purposes

- **To name and reuse string patterns** across a codebase.
- **To parameterize patterns** with generic type variables.
- **To compose complex patterns** from smaller named templates.
- **To document the intended string format** with a descriptive name.

### Syntax Rules and Structure

```typescript
type Pattern<T extends string> = `prefix-${T}-suffix`;
```

**Syntax rules.**

- Generic parameters are typically constrained to `string` (or a literal union).
- The declaration may use intrinsic utilities and conditional types.
- The alias is erased and produces no JavaScript.

**Constraints and limitations.**

- Unconstrained generic parameters may produce unintended combinations.
- Deeply nested declarations can affect readability and performance.

### Annotated Complete Step by Step Code Examples

#### Example 1: Generic template declaration

```typescript
// example_template_declaration.ts
type Prefixed<T extends string> = `app-${T}`;

type Page = "home" | "about" | "contact";
type AppPage = Prefixed<Page>;
// "app-home" | "app-about" | "app-contact"

const page: AppPage = "app-about";
console.log(page); // Expected: app-about
```

**Expected Output:**
```
app-about
```

**Why this output occurs.** The generic `T` is constrained to `string`, and the union `Page` is distributed into the pattern, producing three prefixed literals.

---

### Real-World Cases

- **Route prefixes**: `type ApiRoute<T extends string> = \`/api/${T}\``.
- **Environment variable prefixes**: `type EnvKey<T extends string> = \`APP_${Uppercase<T>}\``.

---

## 4. Template Literal Type Usage

### Definitions

**Core Definition.** **Template literal type usage** refers to applying template literal types in annotations, function parameters, return types, mapped types, and conditional types to enforce string patterns and derive new types.

**Technical Definition.** Template literal types may be used wherever a type is expected. They are commonly used as **parameter types** to constrain string inputs, as **return types** to derive output strings, and inside **mapped types** with `as` clauses for key remapping.

**Beginner-Friendly Explanation.** You use template literal types **anywhere you want to enforce or derive a string pattern**—function arguments, object properties, mapped type keys, and conditional type branches.

### Purposes

- **To constrain string inputs** to a permitted pattern.
- **To derive output string types** from input types.
- **To remap keys in mapped types** with `as` clauses.
- **To pattern-match and extract substrings** with `infer`.

### Syntax Rules and Structure

**Parameter constraint:**

```typescript
function on<E extends string>(event: `on${Capitalize<E>}`): void { ... }
```

**Mapped type key remapping:**

```typescript
type Getters<T> = { [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K] };
```

**Conditional extraction:**

```typescript
type ExtractParam<T extends string> = T extends `${string}:${infer P}/${string}` ? P : never;
```

**Syntax rules.**

- Template literal types in parameter positions constrain callers.
- In mapped types, `as` clauses may use template literal types to rename keys.
- Conditional types with `infer` can decompose strings.

**Constraints and limitations.**

- Parameter constraints with large unions can slow type checking.
- `infer` extraction requires careful pattern ordering.

### Annotated Complete Step by Step Code Examples

#### Example 1: Function parameter constraint

```typescript
// example_usage_param.ts
type EventName = "click" | "focus" | "blur";
type Handler = `on${Capitalize<EventName>}`;

function registerHandler(name: Handler) {
  console.log(`Registered ${name}`);
}

registerHandler("onClick"); // Expected: Registered onClick
// registerHandler("onHover"); // ❌ Error
```

**Expected Output:**
```
Registered onClick
```

**Why this output occurs.** The `Handler` type only permits `"onClick"`, `"onFocus"`, and `"onBlur"`, so invalid handler names are rejected.

---

#### Example 2: Key remapping with `as`

```typescript
// example_usage_remap.ts
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }

const getters: PersonGetters = {
  getName: () => "Ada",
  getAge: () => 36,
};

console.log(getters.getName()); // Expected: Ada
```

**Expected Output:**
```
Ada
```

**Why this output occurs.** The `as` clause remaps each key to `` `get${Capitalize<K>}` ``, producing `getName` and `getAge`.

---

### Real-World Cases

- **Event emitters**: constraining event names to `on${Capitalize<Event>}`.
- **CSS-in-JS**: constraining style values to `${number}${Unit}`.
- **API clients**: constraining route strings to `/api/${string}`.

---

## 5. String Composition

### Definitions

**Core Definition.** **String composition** is the use of template literal types to **concatenate literal types**—prefixes, suffixes, and multiple unions—into new string literal types.

**Technical Definition.** Composition combines literal text with interpolated types. Multiple unions in interpolation positions produce the **Cartesian product** of their members. Intrinsic string utilities (`Uppercase`, `Lowercase`, `Capitalize`, `Uncapitalize`) transform interpolated literals before composition.

**Beginner-Friendly Explanation.** String composition is **gluing string types together**. You take existing literal unions, prefix or suffix them, and TypeScript produces every combination as a new union.

### Purposes

- **To build prefixed or suffixed literal unions** from base unions.
- **To generate naming conventions** (getters, setters, environment keys) systematically.
- **To compose multi-part identifiers** from independent dimensions.
- **To apply case transformations** during composition.

### Syntax Rules and Structure

```typescript
type Prefixed<T extends string> = `prefix_${T}`;
type Multi<A extends string, B extends string> = `${A}-${B}`;
type CaseTransformed<T extends string> = `get${Capitalize<T>}`;
```

**Syntax rules.**

- Multiple interpolations multiply their unions.
- Intrinsic utilities may be nested inside interpolations.
- Literal text may appear anywhere around interpolations.

**Constraints and limitations.**

- Combinatorial explosion occurs when composing large unions.
- Symbol keys are dropped by template literal composition unless handled separately.

### Annotated Complete Step by Step Code Examples

#### Example 1: Prefix and suffix composition

```typescript
// example_compose.ts
type Base = "user" | "post" | "comment";

type Prefixed = `api_${Base}`;
// "api_user" | "api_post" | "api_comment"

type Suffixed = `${Base}_id`;
// "user_id" | "post_id" | "comment_id"

type Both = `api_${Base}_id`;
// "api_user_id" | "api_post_id" | "api_comment_id"

const key: Both = "api_user_id";
console.log(key); // Expected: api_user_id
```

**Expected Output:**
```
api_user_id
```

**Why this output occurs.** Each base member is composed with the prefix `"api_"` and suffix `"_id"`, producing three composed literals.

---

#### Example 2: Case transformation during composition

```typescript
// example_compose_case.ts
type Endpoint = "userProfile" | "orderHistory";
type EnvKey = `API_${Uppercase<Endpoint>}_URL`;
// "API_USERPROFILE_URL" | "API_ORDERHISTORY_URL"

const key: EnvKey = "API_USERPROFILE_URL";
console.log(key); // Expected: API_USERPROFILE_URL
```

**Expected Output:**
```
API_USERPROFILE_URL
```

**Why this output occurs.** `Uppercase` transforms each endpoint literal before composition, producing uppercase environment variable keys.

---

### Real-World Cases

- **Environment keys**: `API_${Uppercase<Endpoint>}_URL`.
- **CSS class prefixes**: `btn-${Color}-${Size}`.
- **Database column names**: `${Table}_${Column}`.

---

## 6. Pattern Modeling

### Definitions

**Core Definition.** **Pattern modeling** is the use of template literal types with **conditional types and `infer`** to describe and extract **structured string shapes**—URLs, routes, CSS values, SQL fragments, and other formatted strings.

**Technical Definition.** Template literal types can **pattern-match** strings when used in the `extends` clause of a conditional type. The `infer` keyword captures substrings at designated positions. Recursive template literal types can split on delimiters, extract repeated parameters, and parse structured strings into named parts.

**Beginner-Friendly Explanation.** Pattern modeling is **using template literal types to understand the structure of a string**. You can say “this string looks like `/users/:id/posts/:postId`” and have TypeScript extract `"id"` and `"postId"` as types.

### Purposes

- **To extract route parameters** from path patterns.
- **To validate structured strings** (URLs, versions, CSS shorthands).
- **To decompose strings into named pieces** at the type level.
- **To build type-safe parsers** without runtime code.

### Syntax Rules and Structure

**Basic extraction:**

```typescript
type ExtractParam<T extends string> =
  T extends `${string}:${infer Param}/${string}` ? Param : never;
```

**Recursive extraction:**

```typescript
type RouteParam<T extends string> =
  T extends `${infer _Start}:${infer Param}/${infer Rest}`
    ? Param | RouteParam<`/${Rest}`>
    : T extends `${infer _Start}:${infer Param}`
      ? Param
      : never;
```

**Syntax rules.**

- `infer` may only appear in the `extends` clause of a conditional type.
- Patterns match from left to right; ordering matters.
- Recursive types require a base case to terminate.

**Constraints and limitations.**

- Recursive template literal types hit instantiation depth limits (~50 levels).
- Greedy matching can produce unexpected splits; use explicit delimiters.

### Annotated Complete Step by Step Code Examples

#### Example 1: Extract route parameters

```typescript
// example_pattern_route.ts
type RouteParam<T extends string> =
  T extends `${infer _Start}:${infer Param}/${infer Rest}`
    ? Param | RouteParam<`/${Rest}`>
    : T extends `${infer _Start}:${infer Param}`
      ? Param
      : never;

type UserRoute = "/users/:userId/posts/:postId";
type Params = RouteParam<UserRoute>;
// "userId" | "postId"

const param: Params = "userId";
console.log(param); // Expected: userId
```

**Expected Output:**
```
userId
```

**Why this output occurs.** The recursive conditional type matches the first `:param/` segment, captures `"userId"`, and recurses on the remainder to capture `"postId"`.

---

#### Example 2: CSS value pattern

```typescript
// example_pattern_css.ts
type CSSUnit = "px" | "em" | "rem" | "%";
type CSSValue = `${number}${CSSUnit}`;

const valid: CSSValue = "100px";
console.log(valid); // Expected: 100px

// const invalid: CSSValue = "100"; // ❌ Error: missing unit.
```

**Expected Output:**
```
100px
```

**Why this output occurs.** The template literal type requires a numeric literal followed by one of the permitted units. Strings without a unit are rejected.

---

### Real-World Cases

- **Route parameter extraction**: `RouteParam<"/users/:id/posts/:postId">`.
- **CSS value validation**: `` `${number}px` | `${number}em` ``.
- **Version string parsing**: `type SemVer = \`${number}.${number}.${number}\``.

---

## 7. Event-Name Types

### Definitions

**Core Definition.** **Event-name types** are template literal types that derive **typed event handler names** from a union of base event names, typically by prefixing with `"on"` and applying `Capitalize`.

**Technical Definition.** Event-name types use the pattern `` `on${Capitalize<EventUnion>}` `` to produce handler names such as `"onClick"`, `"onFocus"`, and `"onBlur"`. They are commonly used in **mapped types** to create handler maps and in **function parameters** to constrain registered handlers.

**Beginner-Friendly Explanation.** Event-name types are **a way to generate valid event handler names automatically**. Instead of typing `"onClick"` by hand and risking typos, you derive it from `"click"`.

### Purposes

- **To enforce valid event handler names** in frameworks and components.
- **To generate handler maps** with keys derived from event unions.
- **To constrain event registration functions** to known events.
- **To avoid typos** in string-based event APIs.

### Syntax Rules and Structure

```typescript
type DOMEvents = "click" | "focus" | "blur";
type EventHandlerName = `on${Capitalize<DOMEvents>}`;
// "onClick" | "onFocus" | "onBlur"
```

**Handler map:**

```typescript
type EventHandlers = { [K in EventHandlerName]?: () => void };
```

**Syntax rules.**

- `Capitalize` is applied to each event name before prefixing.
- The resulting union is used as parameter or mapped-type keys.
- Event names must be string literal types.

**Constraints and limitations.**

- Event names with hyphens (`"mouse-enter"`) require additional transformation.
- Large event unions multiply into many handler names.

### Annotated Complete Step by Step Code Examples

#### Example 1: Handler map

```typescript
// example_event_handlers.ts
type DOMEvents = "click" | "focus" | "blur" | "change";
type EventHandlerName = `on${Capitalize<DOMEvents>}`;

type EventHandlers = {
  [K in EventHandlerName]?: () => void;
};

const handlers: EventHandlers = {
  onClick: () => console.log("clicked"),
  onFocus: () => console.log("focused"),
  // onHover: () => {} // ❌ Error: not a valid handler
};

handlers.onClick?.(); // Expected: clicked
```

**Expected Output:**
```
clicked
```

**Why this output occurs.** The mapped type iterates over the derived handler names, creating optional properties for each valid event. Invalid names are rejected.

---

### Real-World Cases

- **React components**: dynamic event handler props.
- **DOM libraries**: typed `addEventListener` wrappers.
- **Custom event emitters**: typed event registration.

---

## 8. Route and API-Path Types

### Definitions

**Core Definition.** **Route and API-path types** are template literal types that model **URL paths and API routes**, optionally extracting **path parameters** with `infer` and recursive conditional types.

**Technical Definition.** Route types use template literal patterns such as `` `/api/${string}` `` to constrain paths, or recursive extraction patterns to derive parameter unions from route strings like `"/users/:id"`. They enable compile-time validation of route strings and type-safe parameter objects.

**Beginner-Friendly Explanation.** Route types are **string types that describe the shape of a URL**. You can enforce that a path starts with `/api/`, or extract the `:id` parts from a route pattern as a union of parameter names.

### Purposes

- **To validate route strings** at compile time and catch typos.
- **To extract path parameters** as type unions.
- **To build type-safe routers and API clients**.
- **To keep route definitions and parameter types synchronized**.

### Syntax Rules and Structure

**Simple route constraint:**

```typescript
type ApiRoute = `/api/${string}`;
```

**Parameter extraction:**

```typescript
type RouteParam<T extends string> =
  T extends `${infer _Start}:${infer Param}/${infer Rest}`
    ? Param | RouteParam<`/${Rest}`>
    : T extends `${infer _Start}:${infer Param}`
      ? Param
      : never;
```

**Parameter object:**

```typescript
type RouteParams<T extends string> = {
  [K in RouteParam<T>]: string;
};
```

**Syntax rules.**

- Route patterns may use literal prefixes (`/api/`) and string interpolations.
- Parameter extraction requires `infer` and recursion.
- Recursive extraction must have a base case.

**Constraints and limitations.**

- Recursive route parsing can hit recursion limits on very long routes.
- Greedy `infer` may capture more than intended; delimiters must be explicit.

### Annotated Complete Step by Step Code Examples

#### Example 1: API route constraint

```typescript
// example_route_constraint.ts
type ApiRoute = `/api/${string}`;

function fetchApi(route: ApiRoute) {
  console.log(`Fetching ${route}`);
}

fetchApi("/api/users"); // Expected: Fetching /api/users
// fetchApi("/users"); // ❌ Error: missing /api/ prefix.
```

**Expected Output:**
```
Fetching /api/users
```

**Why this output occurs.** The template literal type requires the `/api/` prefix, rejecting paths that do not match.

---

#### Example 2: Route parameter extraction

```typescript
// example_route_params.ts
type RouteParam<T extends string> =
  T extends `${infer _Start}:${infer Param}/${infer Rest}`
    ? Param | RouteParam<`/${Rest}`>
    : T extends `${infer _Start}:${infer Param}`
      ? Param
      : never;

type UserRoute = "/users/:userId/posts/:postId";
type Params = RouteParam<UserRoute>; // "userId" | "postId"

const params: Record<Params, string> = {
  userId: "123",
  postId: "456",
};

console.log(params.postId); // Expected: 456
```

**Expected Output:**
```
456
```

**Why this output occurs.** The recursive conditional type extracts the parameter names from the route pattern and produces a union. The `Record<Params, string>` type requires exactly those keys.

---

### Real-World Cases

- **API clients**: constraining request paths to `/api/${string}`.
- **Routers**: deriving parameter types from route definitions.
- **SDK route maps**: type-safe route builders.

---

## 9. Template Literal Type Best Practices

### Definitions

**Core Definition.** **Template literal type best practices** are guidelines for using these types **safely and performantly**, avoiding **combinatorial explosion**, **recursion limits**, and **unreadable type errors**.

**Technical Definition.** Best practices center on **keeping unions small**, **limiting interpolation dimensions**, **avoiding deep recursion**, **preferring runtime validation for large sets**, **using `as const` objects for route tables**, and **monitoring compiler performance**. Template literal types are **combinatorial by nature**: every interpolated union multiplies the total member count. Large combinations (hundreds or thousands) cause **tsserver lag** and can trigger **TS2590 (union too complex)** or **TS2589 (excessive instantiation depth)**.

**Beginner-Friendly Explanation.** Best practices are **the habits experienced TypeScript developers use** to keep template literal types from slowing down the compiler. The big rules are: keep unions small; do not multiply too many dimensions; do not recurse too deeply; and use runtime validation (like Zod) for large dynamic sets instead of forcing the compiler to enumerate every possibility.

### Purposes

- **To prevent compiler slowdowns** from combinatorial explosion.
- **To avoid recursion limit errors** (TS2589) and union complexity errors (TS2590).
- **To keep type errors readable** and debuggable.
- **To choose the right tool** between template literal types and runtime validation.
- **To maintain editor responsiveness** in large codebases.

### Syntax Rules and Structure

Best practices are conventions that interact with template literal type syntax. The core rules are:

1. **Keep interpolated unions small.** Two unions of three produce nine combinations—fine. Adding a fourth dimension multiplies again. Past a few hundred combinations, type safety is not worth the editor cost.
2. **Limit interpolation dimensions.** Each additional `${...}` position multiplies the member count. Use at most two or three dimensions where possible.
3. **Avoid deep recursion.** Recursive template literal types hit instantiation depth limits around **~50 levels**; keep recursion depth **under 10 levels** where possible.
4. **Prefer runtime validation for large dynamic sets.** For large or user-provided string sets, validate at the boundary with a regex or Zod instead of enumerating all possibilities in the type system.
5. **Use `as const` objects for route tables.** `as const` route maps are more performant and tree-shakeable than large template literal unions.
6. **Widen to `string` when necessary.** If the combinatorial cost is too high, use `string` and validate at runtime.
7. **Monitor `tsc` performance.** Watch for editor lag on every keystroke; it is a sign the type is too expensive.

**Component breakdown (template literal type decision tree):**

```
Do you need to enforce a string pattern?
  ├── Yes → Is the pattern built from small unions (≤ 3 members each, ≤ 3 dimensions)?
  │        ├── Yes → Use a template literal type.
  │        └── No → Is the set dynamic or large?
  │                 ├── Yes → Use `string` + runtime validation (Zod, regex).
  │                 └── No → Consider grouping or branded types.
  └── No → Use plain string literal unions.
```

**Constraints and limitations.**

- Template literal types cannot replace runtime validation for untrusted input.
- Even well-structured patterns can hit limits with enough dimensions.
- Recursive route parsers are convenient but fragile on long routes.

### Annotated Complete Step by Step Code Examples

#### Example 1: Combinatorial explosion

```typescript
// example_best_explosion.ts
// ❌ Large unions multiply into thousands of members.
// type Entity = "user" | "post" | ... (26 members);
// type Action = "created" | "updated" | ... (14 members);
// type Actor = "system" | "admin" | ... (12 members);
// type EventName = `${Entity}.${Action}.by-${Actor}`; // ~4,368 members → lag.

// ✅ Keep unions small and grouped.
type EventName =
  | `user.${"created" | "updated" | "deleted"}`
  | `post.${"created" | "updated"}`;

const event: EventName = "user.created";
console.log(event); // Expected: user.created
```

**Expected Output:**
```
user.created
```

**Why this output occurs.** The union is kept small and grouped by entity, avoiding the combinatorial explosion of multiplying three large dimensions.

---

#### Example 2: Runtime validation for large sets

```typescript
// example_best_runtime.ts
// ✅ Use `string` and validate at the boundary.
type Route = string;

function validateRoute(route: string): asserts route is `/api/${string}` {
  if (!route.startsWith("/api/")) {
    throw new Error("Invalid API route");
  }
}

const route: Route = "/api/users";
validateRoute(route);
console.log(route); // Expected: /api/users
```

**Expected Output:**
```
/api/users
```

**Why this output occurs.** Instead of enumerating every possible route in the type system, the assertion function validates the pattern at runtime, avoiding compiler cost while preserving type safety after validation.

---

#### Example 3: Limiting interpolation dimensions

```typescript
// example_best_dimensions.ts
// ❌ Three dimensions multiply quickly.
// type Class = `btn-${Size}-${Color}-${State}`; // 3 × 3 × 4 = 36

// ✅ Two dimensions are usually enough.
type Class = `btn-${"sm" | "md" | "lg"}-${"primary" | "danger"}`;
// 3 × 2 = 6 members

const cls: Class = "btn-md-primary";
console.log(cls); // Expected: btn-md-primary
```

**Expected Output:**
```
btn-md-primary
```

**Why this output occurs.** Reducing from three dimensions to two keeps the union small and the editor responsive while still providing useful type safety.

---

### Real-World Cases

- **Large event systems**: using runtime validation instead of enumerating thousands of event names.
- **Route tables**: using `as const` objects for performant route maps.
- **CSS-in-JS**: keeping unit unions small (`px | em | rem | %`).
- **SDK design**: grouping template dimensions to avoid combinatorial blowup.

---

## References

- Documentation - Template Literal Types — https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
- Documentation - TypeScript 4.1: Template Literal Types — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#template-literal-types
- Documentation - Mapped Types: Key Remapping via `as` — https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
- Playground Example - String Manipulation with Template Literals — https://www.typescriptlang.org/play#example/template-literal-types
- Steve Kinney - Template Literal Types — https://stevekinney.com/courses/react-typescript/typescript-template-literal-types
- Advanced TypeScript Type Patterns — https://cdn.jsdelivr.net/npm/@su-record/vibe@2.9.40/skills/typescript-advanced-types/rubrics/type-patterns.md
- Template Literal Types Guide — https://www.aiwisdom.dev/articles/typescript/template-literals
- TypeScript Template Literal Types: String Manipulation — https://www.application-architect.com/posts/typescript-template-literal-types-string-manipulation/
- Microsoft/TypeScript Issue #63342 - Type checking complexity with multiple template literals in unions — https://github.com/microsoft/TypeScript/issues/63342
- TypeScript Template Literal Types: When Clever Types Explode — https://www.edge-cases.com