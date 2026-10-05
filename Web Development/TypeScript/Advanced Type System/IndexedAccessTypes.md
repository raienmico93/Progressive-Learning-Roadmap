# TypeScript Indexed Access Types: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
An indexed access type is a TypeScript type operator that extracts the type of a property from another type, using the syntax `T[K]` where `T` is a type and `K` is a key (a literal type, a union of keys, or `number`). It is the type-level equivalent of JavaScript's bracket property access.

**Technical Definition**
Indexed access types, introduced in TypeScript 2.1, use the form `T[K]` to produce the type of the property named `K` in type `T`. When `K` is a literal type (e.g., `"name"`), the result is the type of that specific property. When `K` is a union of literal types, the result is a union of the corresponding property types. When `K` is `number` and `T` is an array or tuple, the result is the element type. When `K` is `keyof T`, the result is the union of all property types. Indexed access types can be nested (`T[K1][K2]`) and combined with generics to enable type-level lookups. They are the foundation for utility types like `Pick`, `Omit`, and `ReturnType`.

**Beginner-Friendly Explanation**
Indexed access types let you look up the type of a property from another type. If you have a `Person` type with `name` and `age`, then `Person["name"]` gives you `string` and `Person["age"]` gives you `number`. It's like using bracket notation on an object, but at the type level. You can use it with arrays (`string[][number]` is `string`), with unions of keys (`Person["name" | "age"]` gives `string | number`), and with `keyof` (`Person[keyof Person]` gives the union of all property types). This is one of the most powerful type-level operators in TypeScript—it's how the standard library's utility types extract property types, function return types, and more.

### Key Characteristics

- **Type-level bracket access**: `T[K]` mirrors JavaScript's `obj[key]` at the type level.
- **Literal key extraction**: `T["prop"]` extracts a specific property's type.
- **Union distribution**: `T[K1 | K2]` produces a union of property types.
- **Array element extraction**: `T[number]` extracts the element type of arrays and tuples.
- **keyof combination**: `T[keyof T]` produces the union of all property types.
- **Nested access**: `T[K1][K2]` enables deep type lookups.
- **Generic-compatible**: `T[K]` where `K extends keyof T` enables type-safe generic lookups.
- **Compile-time only**: Erased at runtime; exists only in the type system.

### Prerequisites

- Basic knowledge of TypeScript object types and interfaces
- Familiarity with union types and literal types
- Understanding of the `keyof` operator
- Familiarity with arrays, tuples, and generics

### Related Programming Areas

- **Type-Level Programming**: Indexed access types are the foundation for utility types and type transformations.
- **Utility Types**: `Pick<T, K>`, `Omit<T, K>`, `ReturnType<T>`, and `Parameters<T>` use indexed access.
- **Key-Based Constraints**: `K extends keyof T` combined with `T[K]` enables type-safe property access.
- **API Design**: Indexed access types enable precise typing of configuration and response structures.
- **Type Inference**: Indexed access types work with `infer` to extract types from complex structures.

### Core Concepts / Features

1. Property Lookup Types (`Type[Key]`) for Extracting Inner Schemas
2. Array and Tuple Element Type Extraction (`Type[number]`)
3. Nested Indexed Access for Deep Configuration Models
4. Using Indexed Access with Dynamic Union Types (`Type[keyof Type]`)


## 1. Property Lookup Types (`Type[Key]`) for Extracting Inner Schemas

### Definitions

**Core Definition**
Property lookup types use indexed access to extract the type of a specific property from an object type. The syntax `Type["propertyName"]` produces the type of that property, enabling precise type extraction from complex structures.

**Technical Definition**
For an object type `T` with property `P` of type `U`, the indexed access type `T[P]` (where `P` is the string literal `"P"`) evaluates to `U`. When `P` is a union of literal types (`"a" | "b"`), the result is a union of the corresponding property types (`T["a"] | T["b"]`). The key can be a literal type, a union of literal types, a generic type parameter constrained to `keyof T`, or the `keyof T` operator itself. Property lookup types are the foundation for extracting "inner schemas"—the types nested within object properties—without manually duplicating them.

**Beginner-Friendly Explanation**
Property lookup types let you extract the type of a specific property. If you have `interface User { name: string; address: { city: string; zip: string } }`, then `User["name"]` is `string` and `User["address"]` is `{ city: string; zip: string }`. You can go deeper: `User["address"]["city"]` is `string`. This is called "extracting inner schemas"—you can pull out nested types without rewriting them. If you have a union of keys, like `User["name" | "address"]`, you get a union of the corresponding types. This is the type-level equivalent of `obj["name"]`, and it's how TypeScript's utility types work under the hood.

### Purposes

- To extract the type of a specific property from an object type.
- To extract nested types without manually duplicating their definitions.
- To build type-safe accessors and transformers that work with any property.
- To serve as the foundation for utility types like `Pick` and `Omit`.
- To enable type-level lookups that stay synchronized with the source type.

### Syntax Rules and Structure

**General Syntax: Property Lookup**

```typescript
type PropertyType = Type["propertyName"];
```

**Component Breakdown**
- `Type`: The object type being accessed.
- `["propertyName"]`: The string literal key.
- `PropertyType`: The extracted property type.

**General Syntax: Union of Keys**

```typescript
type UnionType = Type["key1" | "key2"];
// Equivalent to Type["key1"] | Type["key2"]
```

**Component Breakdown**
- `"key1" | "key2"`: A union of literal keys.
- The result is a union of the corresponding property types.

**General Syntax: Generic Property Lookup**

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

**Component Breakdown**
- `K extends keyof T`: The key is constrained to valid property names.
- `): T[K]`: The return type is the property type at `K`.

**General Syntax: Nested Property Lookup**

```typescript
type City = User["address"]["city"];
```

**Component Breakdown**
- Multiple indexed accesses chain to extract deeply nested types.

**Syntax Rules**

- The key must be a literal type, a union of literal types, or `keyof T`.
- The key can be a generic type parameter constrained to `keyof T`.
- The result is the type of the property at the given key.
- Union keys produce union results.
- Nested access chains multiple lookups.
- Indexed access types work with interfaces, type aliases, and classes.
- The source type must have the key as a public property.

**Constraints and Limitations**

- The key must exist on the type (or be `keyof T`), or the result is a compile error.
- Private and protected members are not accessible via indexed access.
- Index signatures affect the result (see Section 4).
- Indexed access does not perform any transformation—it only extracts.
- Accessing a non-existent key produces a compile error, not `undefined`.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Property Lookup

```typescript
// Step 1: Define an interface.
interface User {
  id: number;
  name: string;
  email: string;
  address: {
    street: string;
    city: string;
    zip: string;
  };
}

// Step 2: Extract simple property types.
type UserId = User["id"];       // number
type UserName = User["name"];   // string
type UserEmail = User["email"]; // string

// Step 3: Extract an inner schema.
type UserAddress = User["address"];
// { street: string; city: string; zip: string }

// Step 4: Extract nested property types.
type UserCity = User["address"]["city"];  // string
type UserZip = User["address"]["zip"];    // string

// Step 5: Use the extracted types.
const userId: UserId = 1;
const userName: UserName = "Alice";
const city: UserCity = "Springfield";

console.log(userId);    // 1
console.log(userName);  // "Alice"
console.log(city);      // "Springfield"

// Step 6: Invalid keys are compile errors.
// type Invalid = User["gender"];  // ❌ Error: Property 'gender' does not exist on type 'User'.
```

**Expected Output:**
```
1
Alice
Springfield
```

**Why This Output Occurs:** `User["id"]` extracts `number`, `User["name"]` extracts `string`, and `User["address"]` extracts the nested address type. `User["address"]["city"]` chains two lookups to extract `string`. The types are precisely extracted from `User` and stay synchronized with it.

#### Example 2: Union of Keys and Generic Lookup

```typescript
// Step 1: Define an interface.
interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
}

// Step 2: Extract a union of property types.
type ProductIdOrName = Product["id" | "name"];
// number | string

type ProductNumeric = Product["id" | "price"];
// number | number = number

// Step 3: Generic property lookup function.
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const product: Product = {
  id: 1,
  name: "Laptop",
  price: 999,
  inStock: true,
};

// Step 4: The return type is precisely inferred for each key.
const id = getProperty(product, "id");       // number
const name = getProperty(product, "name");   // string
const price = getProperty(product, "price"); // number

console.log(id);     // 1
console.log(name);   // "Laptop"
console.log(price);  // 999

// Step 5: Union keys with a generic function.
function getProperties<T, K extends keyof T>(obj: T, keys: K[]): T[K][] {
  return keys.map((key) => obj[key]);
}

const values = getProperties(product, ["id", "name"]);
// (number | string)[]
console.log(values);  // [1, "Laptop"]

// Step 6: Invalid keys are compile errors.
// getProperty(product, "category");  // ❌ Error: "category" is not a key of Product.
```

**Expected Output:**
```
1
Laptop
999
[ 1, 'Laptop' ]
```

**Why This Output Occurs:** `Product["id" | "name"]` produces `number | string`. The generic `getProperty` function uses `K extends keyof T` to constrain the key and `T[K]` to type the return value, so each call returns the precise property type. The `getProperties` function returns an array of the union type.

### Real-World Cases

**Case 1: API Response Typing**
API response types use indexed access to extract nested data types (e.g., `ApiResponse["data"]`), enabling type-safe access to response payloads.

**Case 2: Configuration Models**
Configuration types use indexed access to extract nested settings (e.g., `Config["database"]["host"]`), ensuring type safety across configuration layers.

**Case 3: Form Libraries**
Form libraries use `FormValues["fieldName"]` to type field values, ensuring that validation and submission handlers match the form's shape.

**Case 4: Redux Selectors**
Redux selectors use `RootState["user"]["profile"]` to type the return value of state selectors, ensuring type safety across state slices.

**Case 5: Component Props**
Component libraries use `Props["variant"]` to extract union types for prop values, enabling type-safe component APIs.

**Case 6: Database Schema Types**
ORM types use indexed access to extract entity field types (e.g., `UserEntity["email"]`), ensuring type safety across database operations.


## 2. Array and Tuple Element Type Extraction (`Type[number]`)

### Definitions

**Core Definition**
Array and tuple element type extraction uses the indexed access type `Type[number]` to extract the element type from an array or tuple type. The `number` key represents the numeric indices of the array, and the result is the union of all possible element types.

**Technical Definition**
For an array type `T[]`, the indexed access `T[number]` evaluates to `T` (the element type). For a tuple type `[A, B, C]`, `T[number]` evaluates to `A | B | C` (the union of all element types). The `number` key is a special key that accesses the numeric index signature of the array. For readonly arrays and tuples, the same extraction applies. The pattern is fundamental for extracting element types from arrays of objects, enabling type-safe processing of arrays without duplicating the element type.

**Beginner-Friendly Explanation**
`Type[number]` is how you get the element type of an array or tuple. If you have `string[]`, then `string[][number]` is `string`. If you have a tuple like `[number, string, boolean]`, then `[number, string, boolean][number]` is `number | string | boolean`. This is incredibly useful when you have an array of objects and you want to extract the object type without defining it separately. For example, if you have `const users = [{ name: "Alice" }, { name: "Bob" }]`, then `typeof users[number]` gives you `{ name: string }`. This is a common pattern for extracting types from arrays of literals or configuration data.

### Purposes

- To extract the element type from an array or tuple type.
- To derive object types from arrays of object literals.
- To enable type-safe processing of array elements without duplicating type definitions.
- To extract union types from tuples.
- To work with readonly arrays and tuples.

### Syntax Rules and Structure

**General Syntax: Array Element Extraction**

```typescript
type ElementType = Type[number];
```

**Component Breakdown**
- `Type`: An array or tuple type.
- `[number]`: The numeric index access.
- `ElementType`: The extracted element type.

**General Syntax: Tuple Union Extraction**

```typescript
type TupleUnion = [A, B, C][number];
// A | B | C
```

**Component Breakdown**
- For tuples, `[number]` produces the union of all element types.

**General Syntax: Readonly Array Element Extraction**

```typescript
type ElementType = ReadonlyArray<T>[number];
// T
```

**Component Breakdown**
- Works with `ReadonlyArray<T>` and `readonly T[]`.

**General Syntax: Element Extraction from Array of Objects**

```typescript
const users = [
  { id: 1, name: "Alice" },
  { id: 2, name: "Bob" },
] as const;

type User = typeof users[number];
// { readonly id: 1; readonly name: "Alice" } | { readonly id: 2; readonly name: "Bob" }
```

**Component Breakdown**
- `typeof users`: The array type.
- `[number]`: Extracts the element type (union of both objects).

**Syntax Rules**

- `T[number]` extracts the element type of arrays and tuples.
- For arrays, the result is the single element type.
- For tuples, the result is the union of all element types.
- Works with `ReadonlyArray<T>` and `readonly T[]`.
- Works with `typeof` to extract element types from value arrays.
- The `number` key is a special key for numeric index access.

**Constraints and Limitations**

- `T[number]` on a non-array type produces a compile error (unless the type has a numeric index signature).
- For tuples with `as const`, the extracted type is a union of the readonly literal objects.
- The extraction does not preserve tuple order—it produces a union.
- Empty tuple types (`[]`) produce `never` when accessed with `[number]`.
- Array methods are not included in the extraction—only the element type.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Array and Tuple Element Extraction

```typescript
// Step 1: Define an array type.
type StringArray = string[];
type StringElement = StringArray[number];  // string

type NumberArray = number[];
type NumberElement = NumberArray[number];  // number

// Step 2: Define a tuple type.
type MixedTuple = [string, number, boolean];
type TupleUnion = MixedTuple[number];
// string | number | boolean

// Step 3: Extract from readonly arrays.
type ReadonlyArrayType = readonly string[];
type ReadonlyElement = ReadonlyArrayType[number];  // string

// Step 4: Use the extracted types.
const str: StringElement = "hello";
const num: NumberElement = 42;
const mixed: TupleUnion = true;

console.log(str);    // "hello"
console.log(num);    // 42
console.log(mixed);  // true

// Step 5: Extract element type from an array of objects.
const users = [
  { id: 1, name: "Alice", role: "admin" },
  { id: 2, name: "Bob", role: "user" },
  { id: 3, name: "Charlie", role: "user" },
];

type User = typeof users[number];
// { id: number; name: string; role: string }

function printUser(user: User): void {
  console.log(`${user.name} (${user.role})`);
}

users.forEach(printUser);
// "Alice (admin)"
// "Bob (user)"
// "Charlie (user)"
```

**Expected Output:**
```
hello
42
true
Alice (admin)
Bob (user)
Charlie (user)
```

**Why This Output Occurs:** `StringArray[number]` extracts `string`, `MixedTuple[number]` extracts `string | number | boolean`, and `typeof users[number]` extracts the union of the object types in the `users` array.

#### Example 2: Element Extraction with `as const`

```typescript
// Step 1: Define a readonly tuple of statuses.
const statuses = ["active", "inactive", "pending"] as const;
type Status = typeof statuses[number];
// "active" | "inactive" | "pending"

// Step 2: Use the extracted union.
function setStatus(status: Status): void {
  console.log(`Status: ${status}`);
}

setStatus("active");    // "Status: active"
// setStatus("deleted");  // ❌ Error: "deleted" is not assignable to Status.

// Step 3: Extract from an array of configuration objects.
const environments = [
  { name: "development", url: "http://localhost:3000", debug: true },
  { name: "production", url: "https://api.example.com", debug: false },
] as const;

type Environment = typeof environments[number];
// Union of both readonly environment objects.

// Step 4: Extract specific property types from the element union.
type EnvironmentName = typeof environments[number]["name"];
// "development" | "production"

type EnvironmentUrl = typeof environments[number]["url"];
// "http://localhost:3000" | "https://api.example.com"

// Step 5: Use the extracted types.
function getEnvironment(name: EnvironmentName): Environment {
  const env = environments.find((e) => e.name === name);
  if (!env) throw new Error(`Unknown environment: ${name}`);
  return env;
}

const prod = getEnvironment("production");
console.log(prod.url);    // "https://api.example.com"
console.log(prod.debug);  // false

// Step 6: Invalid names are compile errors.
// getEnvironment("qa");  // ❌ Error: "qa" is not assignable to EnvironmentName.

console.log("Element extraction complete.");
```

**Expected Output:**
```
Status: active
https://api.example.com
false
Element extraction complete.
```

**Why This Output Occurs:** `typeof statuses[number]` extracts the union of literal strings. `typeof environments[number]` extracts the union of environment objects. Adding `["name"]` extracts the union of name literals. The `as const` assertion preserves literal types, enabling precise extraction.

### Real-World Cases

**Case 1: Configuration Arrays**
Configuration arrays use `typeof configs[number]` to derive the configuration object type, ensuring type safety across configuration entries.

**Case 2: Route Definitions**
Routing libraries use `typeof routes[number]` to derive route types from route definition arrays.

**Case 3: Design Tokens**
Design systems use `typeof tokens[number]` to derive token types from token arrays, ensuring type safety across the design system.

**Case 4: Redux Action Creators**
Redux action creators defined as arrays use `typeof actions[number]` to derive the union of action types.

**Case 5: Form Field Definitions**
Form field definitions stored as arrays use `typeof fields[number]` to derive the union of field types.

**Case 6: Test Case Data**
Test case data arrays use `typeof testCases[number]` to derive the test case type, ensuring type safety across tests.


## 3. Nested Indexed Access for Deep Configuration Models

### Definitions

**Core Definition**
Nested indexed access chains multiple indexed access operations to extract deeply nested types from complex configuration models. The syntax `T[K1][K2][K3]` traverses multiple levels of nested properties to extract the type at the deepest level.

**Technical Definition**
Nested indexed access applies the indexed access operator repeatedly: `T[K1]` extracts the type at `K1`, then `T[K1][K2]` extracts the type at `K2` within that type, and so on. Each level must be a valid property access on the previous result. The pattern is essential for extracting types from deeply nested configuration objects, API responses, and domain models. Nested access can be combined with generics to enable type-safe deep property access functions, and with `keyof` to constrain each level of the lookup.

**Beginner-Friendly Explanation**
Nested indexed access lets you extract types from deep within an object. If you have `Config["database"]["connection"]["host"]`, you're going three levels deep to extract the type of the host. This is essential for configuration models where settings are nested, and for API responses where data is deeply structured. You can write `T["a"]["b"]["c"]` to get the type at that path. When combined with generics, you can write functions that safely access deeply nested properties, with TypeScript checking each level.

### Purposes

- To extract types from deeply nested configuration objects.
- To enable type-safe access to nested API response data.
- To build generic deep-property-access functions.
- To keep nested types synchronized with their source types.
- To support type-safe deep configuration validation.

### Syntax Rules and Structure

**General Syntax: Nested Indexed Access**

```typescript
type DeepType = T[K1][K2][K3];
```

**Component Breakdown**
- `T[K1]`: First-level access.
- `[K2]`: Second-level access on the result of `T[K1]`.
- `[K3]`: Third-level access.

**General Syntax: Generic Deep Access**

```typescript
function getDeep<T, K1 extends keyof T, K2 extends keyof T[K1]>(
  obj: T,
  key1: K1,
  key2: K2
): T[K1][K2] {
  return obj[key1][key2];
}
```

**Component Breakdown**
- `K1 extends keyof T`: First-level key constraint.
- `K2 extends keyof T[K1]`: Second-level key constraint on the first result.
- `): T[K1][K2]`: The deeply extracted type.

**General Syntax: Path-Based Access with Tuple Keys**

```typescript
function getPath<T, K1 extends keyof T, K2 extends keyof T[K1]>(
  obj: T,
  path: [K1, K2]
): T[K1][K2] {
  return obj[path[0]][path[1]];
}
```

**Component Breakdown**
- `path: [K1, K2]`: A tuple of keys representing the path.
- `): T[K1][K2]`: The type at the path.

**Syntax Rules**

- Each level of indexed access must be valid on the previous result.
- The chain can be arbitrarily deep.
- Each level can use literal keys, union keys, or generic type parameters.
- Nested access can be combined with `keyof` at each level.
- Generic constraints must chain: `K2 extends keyof T[K1]`.
- Tuple-based paths enable flexible deep access.

**Constraints and Limitations**

- Each level must be valid, or the entire chain fails.
- Deeply nested access produces complex type errors.
- Generic chains become unwieldy beyond 3-4 levels.
- The compiler may struggle with very deep or recursive indexed access.
- Path-based access requires explicit tuple types for each path length.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Deep Configuration Model Extraction

```typescript
// Step 1: Define a deeply nested configuration type.
interface AppConfig {
  server: {
    host: string;
    port: number;
    ssl: {
      enabled: boolean;
      cert: string;
      key: string;
    };
  };
  database: {
    connection: {
      url: string;
      poolSize: number;
      credentials: {
        username: string;
        password: string;
      };
    };
  };
}

// Step 2: Extract types at various depths.
type ServerHost = AppConfig["server"]["host"];       // string
type ServerPort = AppConfig["server"]["port"];       // number
type SslConfig = AppConfig["server"]["ssl"];         // { enabled: boolean; cert: string; key: string }
type SslEnabled = AppConfig["server"]["ssl"]["enabled"];  // boolean

type DbCredentials = AppConfig["database"]["connection"]["credentials"];
// { username: string; password: string }

type DbUsername = AppConfig["database"]["connection"]["credentials"]["username"];
// string

// Step 3: Use the extracted types.
const host: ServerHost = "localhost";
const port: ServerPort = 3000;
const ssl: SslConfig = { enabled: true, cert: "cert.pem", key: "key.pem" };
const username: DbUsername = "admin";

console.log(host);     // "localhost"
console.log(port);     // 3000
console.log(ssl);      // { enabled: true, cert: 'cert.pem', key: 'key.pem' }
console.log(username); // "admin"

// Step 4: Invalid paths are compile errors.
// type Invalid = AppConfig["server"]["invalid"];  // ❌ Error: 'invalid' does not exist.
// type Invalid2 = AppConfig["server"]["host"]["length"];
// ❌ Error: string does not have property 'length' as a type.
```

**Expected Output:**
```
localhost
3000
{ enabled: true, cert: 'cert.pem', key: 'key.pem' }
admin
```

**Why This Output Occurs:** Each level of indexed access extracts the type at that depth. `AppConfig["server"]["host"]` extracts `string`, and `AppConfig["database"]["connection"]["credentials"]["username"]` extracts `string` from four levels deep. The types stay synchronized with the configuration structure.

#### Example 2: Generic Deep Access with Type Safety

```typescript
// Step 1: Define a nested configuration type.
interface Config {
  api: {
    baseUrl: string;
    version: string;
  };
  auth: {
    token: string;
    expiry: number;
  };
}

// Step 2: Generic deep access function for two levels.
function getDeep<T, K1 extends keyof T, K2 extends keyof T[K1]>(
  obj: T,
  key1: K1,
  key2: K2
): T[K1][K2] {
  return (obj[key1] as any)[key2];
}

const config: Config = {
  api: { baseUrl: "https://api.example.com", version: "v1" },
  auth: { token: "token-123", expiry: 3600 },
};

// Step 3: The return type is precisely inferred.
const baseUrl = getDeep(config, "api", "baseUrl");  // string
const version = getDeep(config, "api", "version");  // string
const token = getDeep(config, "auth", "token");     // string
const expiry = getDeep(config, "auth", "expiry");   // number

console.log(baseUrl);  // "https://api.example.com"
console.log(version);  // "v1"
console.log(token);    // "token-123"
console.log(expiry);   // 3600

// Step 4: Invalid paths are compile errors.
// getDeep(config, "api", "invalid");  // ❌ Error: "invalid" is not a key of { baseUrl: string; version: string }.
// getDeep(config, "invalid", "baseUrl");  // ❌ Error: "invalid" is not a key of Config.

// Step 5: Generic deep access for three levels.
function getDeep3<
  T,
  K1 extends keyof T,
  K2 extends keyof T[K1],
  K3 extends keyof T[K1][K2]
>(obj: T, k1: K1, k2: K2, k3: K3): T[K1][K2][K3] {
  return ((obj[k1] as any)[k2] as any)[k3];
}

interface DeepConfig {
  server: {
    ssl: {
      cert: string;
      key: string;
    };
  };
}

const deepConfig: DeepConfig = {
  server: { ssl: { cert: "cert.pem", key: "key.pem" } },
};

const cert = getDeep3(deepConfig, "server", "ssl", "cert");  // string
console.log(cert);  // "cert.pem"

console.log("Deep access complete.");
```

**Expected Output:**
```
https://api.example.com
v1
token-123
3600
cert.pem
Deep access complete.
```

**Why This Output Occurs:** The generic `getDeep` function chains constraints (`K2 extends keyof T[K1]`) to ensure each level is valid. The return type `T[K1][K2]` provides the precise type at the path. The three-level version extends the pattern with another constraint and lookup.

### Real-World Cases

**Case 1: Application Configuration**
Applications with deeply nested configuration (server, database, auth, features) use nested indexed access to extract types for each configuration section.

**Case 2: API Response Models**
API responses with nested data (user → profile → address → coordinates) use nested indexed access to type deeply nested data.

**Case 3: Redux State Trees**
Redux state trees with deeply nested slices use nested indexed access to type selectors and reducers.

**Case 4: Database Schema Types**
ORMs use nested indexed access to type entity relationships (e.g., `Order["customer"]["address"]["country"]`).

**Case 5: GraphQL Response Types**
GraphQL responses with deeply nested query results use nested indexed access to type the response structure.

**Case 6: Design System Tokens**
Design systems use nested indexed access to type token hierarchies (e.g., `Theme["colors"]["primary"]["hover"]`).


## 4. Using Indexed Access with Dynamic Union Types (`Type[keyof Type]`)

### Definitions

**Core Definition**
The pattern `Type[keyof Type]` combines indexed access with the `keyof` operator to produce the union of all property types of a type. This "value union" pattern is essential for working with dynamic objects, configuration values, and discriminated unions where the exact key is not known.

**Technical Definition**
For a type `T`, `keyof T` produces the union of its property names, and `T[keyof T]` produces the union of its property types. This is because indexed access distributes over union keys: `T["a" | "b"]` is `T["a"] | T["b"]`. The pattern is used to extract the "value type" of an object—the union of all possible values that can appear in its properties. It is fundamental for building generic utilities that operate on object values, and for typing functions that accept any property value from an object. When combined with index signatures, `T[keyof T]` includes the index signature's value type.

**Beginner-Friendly Explanation**
`Type[keyof Type]` gives you the union of all property types in a type. If you have `interface Config { host: string; port: number; debug: boolean }`, then `keyof Config` is `"host" | "port" | "debug"`, and `Config[keyof Config]` is `string | number | boolean`—the union of all value types. This is called the "value union" pattern. It's useful when you have a function that accepts any value from an object, or when you want to extract the union of all possible values. For example, a function that updates any config value would accept `Config[keyof Config]` as its value parameter.

### Purposes

- To extract the union of all property types from an object type.
- To type functions that accept any property value from an object.
- To work with dynamic objects where the exact key is not known.
- To build generic utilities that operate on object values.
- To extract the value type of index signatures.

### Syntax Rules and Structure

**General Syntax: Value Union**

```typescript
type ValueUnion = Type[keyof Type];
```

**Component Breakdown**
- `keyof Type`: The union of property names.
- `Type[keyof Type]`: The union of property types.

**General Syntax: Value Union with Index Signature**

```typescript
interface StringMap {
  [key: string]: string;
}

type StringMapValues = StringMap[keyof StringMap];
// string (from the index signature)
```

**Component Breakdown**
- `keyof StringMap`: `string | number`.
- `StringMap[string | number]`: `string`.

**General Syntax: Generic Value Access**

```typescript
function getAnyValue<T>(obj: T, key: keyof T): T[keyof T] {
  return obj[key];
}
```

**Component Breakdown**
- `key: keyof T`: Accepts any valid key.
- `): T[keyof T]`: Returns the union of all value types.

**General Syntax: Value Union for Function Parameters**

```typescript
function updateValue<T>(obj: T, key: keyof T, value: T[keyof T]): void {
  obj[key] = value as T[keyof T];
}
```

**Component Breakdown**
- `value: T[keyof T]`: Accepts any value that could appear in the object.

**Syntax Rules**

- `keyof T` produces the union of property names.
- `T[keyof T]` produces the union of property types.
- The pattern distributes over union keys: `T["a" | "b"]` is `T["a"] | T["b"]`.
- Index signatures contribute their value type to the union.
- The pattern works with generic types (`T[keyof T]`).
- The result is the union of all possible values that can appear in the object's properties.

**Constraints and Limitations**

- The union may be very wide (e.g., `string | number | boolean | object`).
- Assigning to `T[keyof T]` requires a type assertion in some cases.
- The pattern does not preserve the relationship between specific keys and values.
- With index signatures, the union includes the index signature's value type.
- The pattern does not work with `never` types (produces `never`).

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Basic Value Union

```typescript
// Step 1: Define an interface.
interface Config {
  host: string;
  port: number;
  debug: boolean;
  tags: string[];
}

// Step 2: Extract the value union.
type ConfigKey = keyof Config;
// "host" | "port" | "debug" | "tags"

type ConfigValue = Config[keyof Config];
// string | number | boolean | string[]

// Step 3: Use the value union in a function.
function printValue(value: ConfigValue): void {
  if (Array.isArray(value)) {
    console.log(`Array: ${value.join(", ")}`);
  } else {
    console.log(`Value: ${value}`);
  }
}

printValue("localhost");      // "Value: localhost"
printValue(3000);             // "Value: 3000"
printValue(true);             // "Value: true"
printValue(["a", "b", "c"]);  // "Array: a, b, c"

// Step 4: Generic value access.
function getValue<T>(obj: T, key: keyof T): T[keyof T] {
  return obj[key];
}

const config: Config = {
  host: "localhost",
  port: 3000,
  debug: true,
  tags: ["dev", "local"],
};

const host = getValue(config, "host");   // string | number | boolean | string[]
const port = getValue(config, "port");   // string | number | boolean | string[]
console.log(host);  // "localhost"
console.log(port);  // 3000

// Step 5: Invalid keys are compile errors.
// getValue(config, "invalid");  // ❌ Error: "invalid" is not a key of Config.

console.log("Value union extraction complete.");
```

**Expected Output:**
```
Value: localhost
Value: 3000
Value: true
Array: a, b, c
localhost
3000
Value union extraction complete.
```

**Why This Output Occurs:** `Config[keyof Config]` produces the union of all property types: `string | number | boolean | string[]`. The `printValue` function accepts any value from the union. The generic `getValue` function returns the union type, not the specific property type (because the key is `keyof T`, not a specific literal).

#### Example 2: Value Union with Index Signatures and Dynamic Objects

```typescript
// Step 1: Define an object with an index signature.
interface DynamicConfig {
  [key: string]: string | number | boolean;
}

// Step 2: Extract the value union.
type DynamicValue = DynamicConfig[keyof DynamicConfig];
// string | number | boolean

// Step 3: Define a function that accepts dynamic config values.
function setDynamicValue(config: DynamicConfig, key: string, value: DynamicValue): void {
  config[key] = value;
}

const config: DynamicConfig = {};
setDynamicValue(config, "host", "localhost");
setDynamicValue(config, "port", 3000);
setDynamicValue(config, "debug", true);

console.log(config);  // { host: 'localhost', port: 3000, debug: true }

// Step 4: Value union with a discriminated union.
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "rectangle"; width: number; height: number };

type ShapeValue = Shape[keyof Shape];
// "circle" | "square" | "rectangle" | number

// Step 5: Extract only the discriminant values.
type ShapeKind = Shape["kind"];
// "circle" | "square" | "rectangle"

function createShape(kind: ShapeKind): Shape {
  switch (kind) {
    case "circle": return { kind: "circle", radius: 5 };
    case "square": return { kind: "square", side: 4 };
    case "rectangle": return { kind: "rectangle", width: 3, height: 6 };
  }
}

const circle = createShape("circle");
console.log(circle);  // { kind: 'circle', radius: 5 }

// Step 6: Value union for records.
type UserRecord = Record<string, { name: string; age: number }>;
type UserRecordValue = UserRecord[keyof UserRecord];
// { name: string; age: number }

const users: UserRecord = {
  alice: { name: "Alice", age: 30 },
  bob: { name: "Bob", age: 25 },
};

const firstUser: UserRecordValue = users["alice"];
console.log(firstUser.name);  // "Alice"

console.log("Dynamic value union complete.");
```

**Expected Output:**
```
{ host: 'localhost', port: 3000, debug: true }
{ kind: 'circle', radius: 5 }
Alice
Dynamic value union complete.
```

**Why This Output Occurs:** `DynamicConfig[keyof DynamicConfig]` produces `string | number | boolean` from the index signature. `Shape[keyof Shape]` produces the union of all property types across the discriminated union. `UserRecord[keyof UserRecord]` produces the record value type.

### Real-World Cases

**Case 1: Configuration Update Functions**
Config update functions use `Config[keyof Config]` to accept any configuration value, ensuring type safety across all settings.

**Case 2: Redux Reducers**
Redux reducers use `State[keyof State]` to type state values when updating arbitrary state slices.

**Case 3: Form Field Validators**
Form validators use `FormValues[keyof FormValues]` to type field values across a heterogeneous form.

**Case 4: Dynamic Object Builders**
Builders that create objects with dynamic keys use `T[keyof T]` to type the values being set.

**Case 5: Event Payload Types**
Event systems use `EventMap[keyof EventMap]` to type the union of all event payloads.

**Case 6: API Parameter Types**
API clients use `Params[keyof Params]` to type the union of all parameter values.

**Case 7: Localization Systems**
i18n systems use `Translations[keyof Translations]` to type the union of all translation values.

---

## References

- TypeScript Handbook: Indexed Access Types — https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
- TypeScript Handbook: Keyof Type Operator — https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- TypeScript Handbook: Typeof Type Operator — https://www.typescriptlang.org/docs/handbook/2/typeof-types.html
- TypeScript 2.1 Release Notes (keyof and Lookup Types) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html
- TypeScript Handbook: Utility Types — https://www.typescriptlang.org/docs/handbook/utility-types.html
- TypeScript Playground: Indexed Access Types — https://www.typescriptlang.org/play/typescript/primitives/indexed-access-types.ts.html
- TypeScript Deep Dive: Indexed Access Types — https://basarat.gitbook.io/typescript/type-system/index-signatures
- Total TypeScript: Indexed Access Types — https://www.totaltypescript.com/workshops/type-transformations/conditional-types-and-infer/indexed-access-types/solution
- Convex TypeScript Guide: Indexed Access Types — https://www.convex.dev/typescript/advanced/type-operators-manipulation/typescript-indexed-access-types
- Stack Overflow: Indexed access types in TypeScript — https://stackoverflow.com/questions/61186358
- Stack Overflow: Difference between `T[number]` and `T[keyof T]` — https://stackoverflow.com/questions/54279954
- MDN: Property Accessors — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Property_accessors
- TypeScript ESLint: no-unnecessary-type-assertion — https://typescript-eslint.io/rules/no-unnecessary-type-assertion/
- Effective TypeScript: Item 14 — Use Type Operations and Generics to Avoid Repeating Yourself