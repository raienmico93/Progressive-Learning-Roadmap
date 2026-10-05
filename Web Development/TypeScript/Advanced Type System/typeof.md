# TypeScript `typeof` in Type Positions: A Comprehensive Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
The `typeof` type operator is a TypeScript feature that allows you to derive a type from a runtime value. While JavaScript's `typeof` returns a string at runtime, TypeScript's `typeof` in a type position returns the TypeScript type of a variable, property, or expression. This bridges the gap between values and types, enabling type definitions to stay synchronized with their runtime sources.

**Technical Definition**
The `typeof` type operator queries the type of an identifier (variable, property, or expression) at compile time. It is only legal to use `typeof` on identifiers (variable names) or their properties—not on arbitrary expressions. The result is a type that reflects the inferred or declared type of the value. When combined with indexed access types (`typeof value[number]`) and `keyof` (`keyof typeof value`), `typeof` enables powerful type derivation patterns. For classes, `typeof ClassName` refers to the type of the constructor function (including static members), while the class name itself (without `typeof`) refers to the instance type. For enums, `typeof EnumObject` gives the type of the enum object, while `EnumObject.Member` gives the literal member type.

**Beginner-Friendly Explanation**
The `typeof` operator in TypeScript lets you say "give me the type of this value." If you have a configuration object and you want to create a type that matches it, you can write `type Config = typeof config`. Now `Config` has the same shape as your config object. If you add a property to the config object, the type updates automatically. This is incredibly useful for keeping types in sync with real values. You can also use `typeof` on functions to get their signature, on classes to get the constructor type, and on enums to get the enum object type. The key rule is that you can only use `typeof` on variable names or their properties—not on arbitrary expressions like function calls.

### Key Characteristics

- **Value-to-type bridge**: Derives types from runtime values, ensuring type/value synchronization.
- **Identifier-only limitation**: Only works on variable names or their properties, not on expressions.
- **Inferred type capture**: Captures the inferred type of a value, including widened literal types.
- **Composable**: Combines with `keyof` (`keyof typeof`) and indexed access (`typeof arr[number]`).
- **Class constructor type**: `typeof ClassName` gives the constructor type (with statics), not the instance type.
- **Enum object type**: `typeof EnumObject` gives the enum object type; `EnumObject.Member` gives the member literal type.
- **Compile-time only**: The `typeof` in a type position is erased at runtime.

### Prerequisites

- Basic knowledge of TypeScript types and type annotations
- Familiarity with variable declarations (`let`, `const`, `var`)
- Understanding of type inference and widening
- Familiarity with `keyof` and indexed access types (`T[K]`)

### Related Programming Areas

- **Type-Level Programming**: `typeof` is a fundamental type query operator
- **Configuration Management**: Keeping types synchronized with configuration values
- **API Design**: Deriving types from existing values and functions
- **Enum Handling**: Distinguishing between enum objects and enum member types
- **Class Reflection**: Querying constructor types and static members

### Core Concepts / Features

1. Deriving Types from Runtime Values, Object Literals, and Arrays
2. Reusing Variable, Function, and Class Constructor Signatures at the Type Level
3. Configuration Type Extraction for Synchronization Between Data and Types
4. `typeof` with Enum Objects Versus Enum Member Types


## 1. Deriving Types from Runtime Values, Object Literals, and Arrays

### Definitions

**Core Definition**
Deriving types from runtime values uses `typeof` to create a type alias from an existing value. This ensures that the type and the value stay synchronized—if the value changes, the type updates automatically. Object literals and arrays are common sources for type derivation.

**Technical Definition**
When `typeof` is applied to a variable holding an object or array, TypeScript produces an object type or array type reflecting the value's inferred type. For `const` declarations, literal types are preserved for primitives but object properties are widened unless `as const` is used. For `let` declarations, types are widened to their base primitives. Indexed access types (`typeof arr[number]`) extract the element type of an array. The `keyof typeof` pattern extracts the keys of an object as a union of literal types. TypeScript 5.0+ improves enum handling, but `typeof` on enums still refers to the enum object type.

**Beginner-Friendly Explanation**
You can use `typeof` to turn a value into a type. If you have a configuration object, `type Config = typeof config` gives you a type with the same shape. If you have an array, `type Element = typeof myArray[number]` gives you the type of the array's elements. This is great for keeping types and values in sync—if you add a property to the config, the type updates automatically. The `keyof typeof` combination is especially useful: it gives you a union of all the keys in an object, which you can use to type function parameters or to ensure you only use valid keys.

### Purposes

- To derive a type from an existing value without manually duplicating its shape.
- To keep types synchronized with configuration objects, arrays, and constants.
- To extract element types from array literals using indexed access.
- To extract keys from objects using `keyof typeof`.
- To reduce duplication between runtime values and type definitions.

### Syntax Rules and Structure

**General Syntax: `typeof` on a Variable**

```typescript
const variableName = { property: "value" };
type VariableType = typeof variableName;
// { property: string }
```

**Component Breakdown**
- `typeof variableName`: Queries the type of the variable.
- `type VariableType`: The derived type alias.

**General Syntax: `typeof` on an Array with Indexed Access**

```typescript
const myArray = [{ name: "Alice", age: 30 }];
type ElementType = typeof myArray[number];
// { name: string; age: number }
```

**Component Breakdown**
- `typeof myArray`: The array type.
- `[number]`: Indexed access extracts the element type.

**General Syntax: `keyof typeof`**

```typescript
const config = { apiUrl: "https://api.example.com", timeout: 5000 };
type ConfigKey = keyof typeof config;
// "apiUrl" | "timeout"
```

**Component Breakdown**
- `typeof config`: The object type.
- `keyof`: Extracts the union of property names.

**General Syntax: `typeof` with `as const`**

```typescript
const config = { apiUrl: "https://api.example.com", timeout: 5000 } as const;
type Config = typeof config;
// { readonly apiUrl: "https://api.example.com"; readonly timeout: 5000 }
```

**Component Breakdown**
- `as const`: Preserves literal types and makes properties readonly.
- `typeof config`: Captures the narrow literal types.

**Syntax Rules**

- `typeof` in a type position queries the type of an identifier or its property.
- Only variable names (identifiers) or their properties can be used—not arbitrary expressions.
- `const` declarations preserve literal types for primitives but widen object properties.
- `as const` prevents widening and makes properties readonly.
- `typeof arr[number]` extracts the element type of an array.
- `keyof typeof obj` extracts the keys of an object.
- `typeof` on a `let` variable captures the widened type.

**Constraints and Limitations**

- `typeof` cannot be used on function calls or arbitrary expressions.
- `typeof` on a `let` variable captures the widened type, not the literal type.
- Object properties are widened even for `const` declarations unless `as const` is used.
- `typeof` on a variable with an explicit type annotation captures that annotation, not the inferred type.
- `typeof` on a class gives the constructor type, not the instance type.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Deriving Types from Object Literals

```typescript
// Step 1: Create a configuration object.
const appConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
  environment: "production",
};

// Step 2: Derive a type from the config object.
type AppConfig = typeof appConfig;
// { apiUrl: string; timeout: number; retries: number; environment: string }

// Step 3: Use the derived type.
function createConfig(config: AppConfig): void {
  console.log(`API: ${config.apiUrl}, Timeout: ${config.timeout}`);
}

createConfig(appConfig);  // "API: https://api.example.com, Timeout: 5000"

// Step 4: Derive keys using keyof typeof.
type AppConfigKey = keyof typeof appConfig;
// "apiUrl" | "timeout" | "retries" | "environment"

function getConfigValue(key: AppConfigKey): typeof appConfig[AppConfigKey] {
  return appConfig[key];
}

console.log(getConfigValue("apiUrl"));   // "https://api.example.com"
console.log(getConfigValue("timeout"));  // 5000

// Step 5: Invalid keys are compile errors.
// getConfigValue("invalidKey");  // ❌ Error: "invalidKey" is not assignable to AppConfigKey.

// Step 6: The type stays in sync if the config changes.
// If we add `secure: true`, AppConfig automatically includes `secure: boolean`.
```

**Expected Output:**
```
API: https://api.example.com, Timeout: 5000
https://api.example.com
5000
```

**Why This Output Occurs:** `typeof appConfig` creates a type with the same shape as the config object. `keyof typeof appConfig` gives the union of keys. The type stays synchronized with the value—changing the config object updates the type.

#### Example 2: Deriving Element Types from Arrays

```typescript
// Step 1: Create an array of objects.
const users = [
  { id: 1, name: "Alice", email: "alice@example.com" },
  { id: 2, name: "Bob", email: "bob@example.com" },
];

// Step 2: Derive the element type.
type User = typeof users[number];
// { id: number; name: string; email: string }

// Step 3: Use the derived element type.
function printUser(user: User): void {
  console.log(`User ${user.id}: ${user.name} (${user.email})`);
}

users.forEach(printUser);
// "User 1: Alice (alice@example.com)"
// "User 2: Bob (bob@example.com)"

// Step 4: Derive specific property types.
type UserName = typeof users[number]["name"];
// string

const name: UserName = users[0].name;
console.log(name);  // "Alice"

// Step 5: With as const for literal types.
const statuses = ["active", "inactive", "pending"] as const;
type Status = typeof statuses[number];
// "active" | "inactive" | "pending"

function setStatus(status: Status): void {
  console.log(`Status: ${status}`);
}

setStatus("active");    // "Status: active"
// setStatus("deleted");  // ❌ Error: "deleted" is not assignable to Status.
```

**Expected Output:**
```
User 1: Alice (alice@example.com)
User 2: Bob (bob@example.com)
Alice
Status: active
```

**Why This Output Occurs:** `typeof users[number]` extracts the element type of the array. `typeof users[number]["name"]` extracts the type of the `name` property. The `as const` assertion preserves literal types, enabling the `Status` union.

### Real-World Cases

**Case 1: Configuration Management**
Application configuration objects use `typeof` to derive types, ensuring that configuration types stay synchronized with their values.

**Case 2: API Response Typing**
API response arrays use `typeof response[number]` to extract the element type, enabling type-safe processing of response data.

**Case 3: Design Tokens**
Design systems use `as const` and `typeof` to derive literal types from design token objects, ensuring type safety across the design system.

**Case 4: Route Definitions**
Routing libraries use `typeof routes[number]` to derive route types from route definition arrays.

**Case 5: Redux Action Types**
Redux action creators use `typeof action` to derive action types, ensuring reducers and action creators stay in sync.


## 2. Reusing Variable, Function, and Class Constructor Signatures at the Type Level

### Definitions

**Core Definition**
`typeof` can be applied to variables, functions, and classes to reuse their types at the type level. For functions, `typeof` captures the function's signature (parameters and return type). For classes, `typeof ClassName` captures the constructor type, including static members.

**Technical Definition**
When `typeof` is applied to a function, it produces the function type, which can be used with `ReturnType<T>` and `Parameters<T>` to extract the return type and parameter types. When applied to a class, `typeof ClassName` produces the constructor type, which includes the constructor signature and any static members. The class name itself (without `typeof`) refers to the instance type. The constructor type can be used with `InstanceType<T>` to extract the instance type. For variables, `typeof` captures the variable's type, which can be a primitive, object, array, or function type.

**Beginner-Friendly Explanation**
You can use `typeof` to get the type of a function or class. If you have a function `greet`, then `typeof greet` is the function's type—you can use it to type other functions with the same signature. For classes, `typeof MyClass` gives you the constructor type, which includes the `new` signature and any static members. The class name itself (without `typeof`) gives you the instance type. This is useful for factory functions, dependency injection, and mixins. You can also use `ReturnType<typeof fn>` to get a function's return type without calling it.

### Purposes

- To reuse function signatures at the type level.
- To extract function return types with `ReturnType<typeof fn>`.
- To extract function parameter types with `Parameters<typeof fn>`.
- To reference class constructor types for factories and dependency injection.
- To access static members through the constructor type.

### Syntax Rules and Structure

**General Syntax: `typeof` on a Function**

```typescript
function greet(name: string): string {
  return `Hello, ${name}`;
}

type GreetFn = typeof greet;
// (name: string) => string
```

**Component Breakdown**
- `typeof greet`: The function type.
- `GreetFn`: A type alias for the function type.

**General Syntax: Extracting Return and Parameter Types**

```typescript
type GreetReturn = ReturnType<typeof greet>;
// string

type GreetParams = Parameters<typeof greet>;
// [name: string]
```

**Component Breakdown**
- `ReturnType<typeof greet>`: Extracts the return type.
- `Parameters<typeof greet>`: Extracts the parameter types as a tuple.

**General Syntax: `typeof` on a Class**

```typescript
class User {
  static create(name: string): User {
    return new User(name);
  }
  constructor(public name: string) {}
}

type UserConstructor = typeof User;
// typeof User (includes static `create` and constructor signature)

type UserInstance = InstanceType<typeof User>;
// User
```

**Component Breakdown**
- `typeof User`: The constructor type (includes statics).
- `InstanceType<typeof User>`: The instance type.

**General Syntax: Using `typeof` in Factory Functions**

```typescript
function createInstance<T extends new (...args: any[]) => any>(
  ctor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new ctor(...args);
}
```

**Component Breakdown**
- `T extends new (...args: any[]) => any`: The constructor constraint.
- `ConstructorParameters<T>`: Extracts the constructor parameters.
- `InstanceType<T>`: Extracts the instance type.

**Syntax Rules**

- `typeof functionName` produces the function's type signature.
- `ReturnType<typeof fn>` extracts the return type.
- `Parameters<typeof fn>` extracts the parameter types as a tuple.
- `typeof ClassName` produces the constructor type (including statics).
- `InstanceType<typeof ClassName>` extracts the instance type.
- `ConstructorParameters<typeof ClassName>` extracts the constructor parameter types.
- `typeof` on a class expression (e.g., `typeof class {}`) is supported in TypeScript 4.2+.

**Constraints and Limitations**

- `typeof` on a function gives the function type, not the return type (use `ReturnType` for that).
- `typeof` on a class gives the constructor type, not the instance type.
- `typeof` on a function with overloads captures all overload signatures.
- `typeof` cannot be used on function calls or arbitrary expressions.
- Generic functions have their type parameters preserved in the `typeof` result.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Function Signature Reuse

```typescript
// Step 1: Define a function.
function formatName(first: string, last: string): string {
  return `${first} ${last}`;
}

// Step 2: Derive the function type.
type FormatNameFn = typeof formatName;
// (first: string, last: string) => string

// Step 3: Use the derived type for another function.
const formatNameUpper: FormatNameFn = (first, last) => {
  return `${first} ${last}`.toUpperCase();
};

console.log(formatNameUpper("Alice", "Smith"));  // "ALICE SMITH"

// Step 4: Extract return type.
type FormatNameReturn = ReturnType<typeof formatName>;
// string

const result: FormatNameReturn = formatName("Bob", "Jones");
console.log(result);  // "Bob Jones"

// Step 5: Extract parameter types.
type FormatNameParams = Parameters<typeof formatName>;
// [first: string, last: string]

const params: FormatNameParams = ["Charlie", "Brown"];
console.log(formatName(...params));  // "Charlie Brown"

// Step 6: Use with higher-order functions.
function logResult<T extends (...args: any[]) => any>(
  fn: T,
  ...args: Parameters<T>
): ReturnType<T> {
  const result = fn(...args);
  console.log(`Result: ${result}`);
  return result;
}

logResult(formatName, "Dave", "Wilson");  // "Result: Dave Wilson"
```

**Expected Output:**
```
ALICE SMITH
Bob Jones
Charlie Brown
Result: Dave Wilson
```

**Why This Output Occurs:** `typeof formatName` captures the function signature. `ReturnType` and `Parameters` extract the return and parameter types. The `logResult` function uses `Parameters<T>` and `ReturnType<T>` to type its arguments and return value.

#### Example 2: Class Constructor Type and Factory Pattern

```typescript
// Step 1: Define a class with a static factory method.
class User {
  static create(name: string): User {
    return new User(name);
  }

  constructor(public name: string, public age: number = 0) {}

  greet(): string {
    return `Hello, I'm ${this.name}`;
  }
}

// Step 2: Derive the constructor type.
type UserConstructor = typeof User;
// typeof User (includes constructor and static `create`)

// Step 3: Use the constructor type in a factory.
function createAndGreet(
  ctor: UserConstructor,
  name: string
): string {
  const user = ctor.create(name);
  return user.greet();
}

console.log(createAndGreet(User, "Alice"));  // "Hello, I'm Alice"

// Step 4: Extract the instance type.
type UserInstance = InstanceType<typeof User>;
// User

const user: UserInstance = new User("Bob", 30);
console.log(user.greet());  // "Hello, I'm Bob"

// Step 5: Generic factory with constructor parameters.
function createEntity<T extends new (...args: any[]) => any>(
  ctor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new ctor(...args);
}

const newUser = createEntity(User, "Charlie", 25);
console.log(newUser.greet());  // "Hello, I'm Charlie"
console.log(newUser.age);      // 25

// Step 6: Access static members through the constructor type.
type UserStatic = typeof User;
const factory: UserStatic = User;
const created = factory.create("Dave");
console.log(created.greet());  // "Hello, I'm Dave"
```

**Expected Output:**
```
Hello, I'm Alice
Hello, I'm Bob
Hello, I'm Charlie
25
Hello, I'm Dave
```

**Why This Output Occurs:** `typeof User` gives the constructor type, which includes the static `create` method and the constructor signature. `InstanceType<typeof User>` gives the instance type. The generic factory uses `ConstructorParameters<T>` and `InstanceType<T>` to create typed instances.

### Real-World Cases

**Case 1: Dependency Injection Containers**
DI containers use `typeof ClassName` to reference constructor types, enabling type-safe instantiation of services.

**Case 2: React Component Factories**
Higher-order components use `typeof Component` to type wrapped components, preserving props and static members.

**Case 3: Testing Utilities**
Test utilities use `typeof fn` to type mocks and spies that match the original function's signature.

**Case 4: ORM Entity Factories**
ORMs use `typeof Entity` to type entity constructors, enabling type-safe creation and querying.

**Case 5: Middleware Systems**
Middleware systems use `typeof middleware` to type middleware chains, ensuring type safety across composition.


## 3. Configuration Type Extraction for Synchronization Between Data and Types

### Definitions

**Core Definition**
Configuration type extraction uses `typeof` to derive a type from a configuration object, ensuring that the type and the configuration stay synchronized. When the configuration changes, the derived type updates automatically.

**Technical Definition**
The pattern `type Config = typeof configObject` creates a type alias that mirrors the structure of the configuration object. Combined with `keyof typeof` and `satisfies`, this enables type-safe access to configuration values and validation of configuration shapes. The `satisfies` operator (TypeScript 4.9+) can validate a configuration object against a type without widening it, preserving literal types. The `keyof typeof` pattern extracts the keys of the configuration as a union, which can be used to type function parameters or to ensure that only valid keys are used.

**Beginner-Friendly Explanation**
Configuration type extraction is a pattern for keeping types and configuration objects in sync. Instead of writing a separate interface for your config, you write `type Config = typeof config`. Now the type matches the object exactly. If you add a property to the config, the type updates. You can also use `keyof typeof config` to get a union of all the config keys, which is useful for functions that access config values. The `satisfies` operator lets you validate that a config object matches a type without changing the type of the config itself—it keeps the narrow literal types while ensuring the shape is correct.

### Purposes

- To keep configuration types synchronized with configuration values.
- To avoid duplicating configuration shapes in separate type definitions.
- To extract configuration keys as a union for type-safe access.
- To validate configuration shapes with `satisfies` while preserving literal types.
- To enable type-safe environment-specific configuration.

### Syntax Rules and Structure

**General Syntax: Basic Configuration Type Extraction**

```typescript
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
};

type Config = typeof config;
```

**Component Breakdown**
- `typeof config`: Captures the configuration's type.
- `Config`: The derived type alias.

**General Syntax: Key Extraction with `keyof typeof`**

```typescript
type ConfigKey = keyof typeof config;
// "apiUrl" | "timeout" | "retries"
```

**Component Breakdown**
- `keyof typeof config`: The union of configuration keys.

**General Syntax: Validation with `satisfies`**

```typescript
interface AppConfig {
  apiUrl: string;
  timeout: number;
}

const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
} satisfies AppConfig;
// config retains its inferred type (with retries), but is validated against AppConfig.
```

**Component Breakdown**
- `satisfies AppConfig`: Validates the object against `AppConfig` without widening.
- `config` retains its full inferred type, including `retries`.

**General Syntax: Environment-Specific Configuration**

```typescript
const configurations = {
  development: { apiUrl: "http://localhost:3000", debug: true },
  production: { apiUrl: "https://api.example.com", debug: false },
} as const;

type Environment = keyof typeof configurations;
// "development" | "production"

type Config = typeof configurations[Environment];
// { readonly apiUrl: "http://localhost:3000"; readonly debug: true }
// | { readonly apiUrl: "https://api.example.com"; readonly debug: false }
```

**Component Breakdown**
- `keyof typeof configurations`: The environment union.
- `typeof configurations[Environment]`: The union of all config variants.

**Syntax Rules**

- `typeof configObject` creates a type from a configuration value.
- `keyof typeof configObject` extracts the configuration keys.
- `satisfies` validates against a type without widening.
- `as const` preserves literal types for configuration values.
- Environment-specific configs use `keyof typeof` and indexed access.
- The derived type stays synchronized with the value.

**Constraints and Limitations**

- `typeof` on a `let` config captures the widened type, not the literal type.
- `as const` is needed to preserve literal types in configuration values.
- `satisfies` requires TypeScript 4.9+.
- The derived type does not enforce that the configuration is correct—it only mirrors its shape.
- Configuration changes that remove properties will break consumers of the derived type.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Configuration Extraction and Key Safety

```typescript
// Step 1: Define a configuration object.
const appConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
  features: {
    darkMode: true,
    notifications: false,
  },
} as const;

// Step 2: Derive the configuration type.
type AppConfig = typeof appConfig;
// {
//   readonly apiUrl: "https://api.example.com";
//   readonly timeout: 5000;
//   readonly retries: 3;
//   readonly features: {
//     readonly darkMode: true;
//     readonly notifications: false;
//   };
// }

// Step 3: Extract keys.
type AppConfigKey = keyof typeof appConfig;
// "apiUrl" | "timeout" | "retries" | "features"

// Step 4: Type-safe configuration access.
function getConfigValue<K extends AppConfigKey>(key: K): AppConfig[K] {
  return appConfig[key];
}

console.log(getConfigValue("apiUrl"));   // "https://api.example.com"
console.log(getConfigValue("timeout"));  // 5000

// Step 5: Nested key access.
type FeatureKey = keyof typeof appConfig.features;
// "darkMode" | "notifications"

function getFeature<K extends FeatureKey>(key: K): AppConfig["features"][K] {
  return appConfig.features[key];
}

console.log(getFeature("darkMode"));       // true
console.log(getFeature("notifications"));  // false

// Step 6: Validation with satisfies.
interface BaseConfig {
  apiUrl: string;
  timeout: number;
}

const validatedConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
} satisfies BaseConfig;
// validatedConfig retains `retries` in its type.

console.log(validatedConfig.retries);  // 3

// Step 7: Invalid keys are compile errors.
// getConfigValue("invalidKey");  // ❌ Error: "invalidKey" is not assignable to AppConfigKey.
```

**Expected Output:**
```
https://api.example.com
5000
true
false
3
```

**Why This Output Occurs:** The `as const` assertion preserves literal types in the configuration. `typeof appConfig` creates the `AppConfig` type. `keyof typeof appConfig` extracts the keys. `satisfies` validates against `BaseConfig` without widening, preserving the `retries` property.

#### Example 2: Environment-Specific Configuration

```typescript
// Step 1: Define environment-specific configurations.
const configurations = {
  development: {
    apiUrl: "http://localhost:3000",
    timeout: 1000,
    debug: true,
  },
  staging: {
    apiUrl: "https://staging.example.com",
    timeout: 5000,
    debug: true,
  },
  production: {
    apiUrl: "https://api.example.com",
    timeout: 10000,
    debug: false,
  },
} as const;

// Step 2: Derive the environment type.
type Environment = keyof typeof configurations;
// "development" | "staging" | "production"

// Step 3: Derive the configuration union type.
type Config = typeof configurations[Environment];
// Union of all three config variants.

// Step 4: Get the current environment from an environment variable.
const currentEnv: Environment = "production";

// Step 5: Access the current configuration.
const currentConfig = configurations[currentEnv];
console.log(currentConfig.apiUrl);  // "https://api.example.com"
console.log(currentConfig.debug);   // false

// Step 6: Type-safe configuration getter.
function getConfig<K extends Environment>(env: K): typeof configurations[K] {
  return configurations[env];
}

const devConfig = getConfig("development");
console.log(devConfig.timeout);  // 1000

// Step 7: Invalid environments are compile errors.
// getConfig("qa");  // ❌ Error: "qa" is not assignable to Environment.

// Step 8: Using keyof typeof for environment validation.
function isValidEnvironment(env: string): env is Environment {
  return env in configurations;
}

console.log(isValidEnvironment("production"));  // true
console.log(isValidEnvironment("qa"));          // false
```

**Expected Output:**
```
https://api.example.com
false
1000
true
false
```

**Why This Output Occurs:** `keyof typeof configurations` gives the union of environment names. `typeof configurations[Environment]` gives the union of all configuration variants. The `getConfig` function uses `K extends Environment` to ensure only valid environment names are used. The type guard validates strings against the environment union.

### Real-World Cases

**Case 1: Multi-Environment Applications**
Applications with development, staging, and production configurations use `keyof typeof` to type the environment and ensure type-safe configuration access.

**Case 2: Feature Flag Systems**
Feature flag systems use `typeof` to derive flag types from flag definition objects, ensuring type-safe flag access.

**Case 3: Theme Systems**
Design systems use `as const` and `typeof` to derive theme types from theme objects, ensuring type safety across the design system.

**Case 4: Internationalization (i18n)**
i18n systems use `typeof` to derive translation key types from translation dictionaries, enabling type-safe translation access.

**Case 5: API Route Configuration**
API route configurations use `typeof` to derive route parameter types, ensuring type safety in route handlers.

**Case 6: Database Schema Configuration**
Database schemas defined as configuration objects use `typeof` to derive TypeScript types, keeping schemas and types synchronized.


## 4. `typeof` with Enum Objects Versus Enum Member Types

### Definitions

**Core Definition**
Enums in TypeScript create two distinct type-level entities: the **enum object** (the runtime value) and the **enum member type** (the literal type of each member). The `typeof` operator captures the enum object's type, while individual members can be used directly as types. Understanding this distinction is crucial for correct enum usage.

**Technical Definition**
An enum declaration introduces both a value (the enum object) and a type (the enum type). The enum object is the runtime value that holds the enum members. `typeof EnumObject` produces the type of the enum object, which is an object type with the enum members as properties. The enum type itself (e.g., `Direction`) is a union of its member types (since TypeScript 5.0, all enums are union enums). Each enum member (e.g., `Direction.Up`) has its own literal type, which can be used as a type annotation. `keyof typeof EnumObject` extracts the union of enum member names. `typeof EnumObject[keyof typeof EnumObject]` extracts the union of enum member values.

**Beginner-Friendly Explanation**
Enums are special because they create both a value and a type. The enum object (the value) has properties like `Direction.Up` and `Direction.Down`. The enum type (used as a type annotation) is a union of those members. When you write `typeof Direction`, you get the type of the enum object—the thing with properties. When you write `Direction.Up` as a type, you get the literal type of that specific member. This distinction matters: `typeof Direction` is for the object, while `Direction` (without `typeof`) is the enum type (union of members). `keyof typeof Direction` gives you the union of member names, and `typeof Direction[keyof typeof Direction]` gives the union of member values.

### Purposes

- To distinguish between the enum object (value) and the enum type (type annotation).
- To correctly use `typeof` with enums in type positions.
- To extract enum member names with `keyof typeof`.
- To extract enum member values with `typeof EnumObject[keyof typeof EnumObject]`.
- To avoid common pitfalls when working with enums in generic code.

### Syntax Rules and Structure

**General Syntax: Enum Object Type (`typeof`)**

```typescript
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}

type DirectionObject = typeof Direction;
// {
//   Up: Direction.Up;
//   Down: Direction.Down;
//   Left: Direction.Left;
//   Right: Direction.Right;
// }
```

**Component Breakdown**
- `typeof Direction`: The type of the enum object.
- The object has properties for each enum member.

**General Syntax: Enum Member Type (Direct)**

```typescript
type UpType = Direction.Up;  // Direction.Up (literal type)
// Or use the enum type itself:
type DirectionType = Direction;  // Direction.Up | Direction.Down | Direction.Left | Direction.Right
```

**Component Breakdown**
- `Direction.Up`: The literal type of the `Up` member.
- `Direction`: The union of all member types.

**General Syntax: Enum Member Names (`keyof typeof`)**

```typescript
type DirectionKey = keyof typeof Direction;
// "Up" | "Down" | "Left" | "Right"
```

**Component Breakdown**
- `keyof typeof Direction`: The union of enum member names.

**General Syntax: Enum Member Values (`typeof[][keyof typeof]`)**

```typescript
type DirectionValue = typeof Direction[keyof typeof Direction];
// "UP" | "DOWN" | "LEFT" | "RIGHT"
```

**Component Breakdown**
- `typeof Direction[keyof typeof Direction]`: The union of enum member values.

**General Syntax: Enum Type (Without `typeof`)**

```typescript
function move(direction: Direction): void {
  // direction is Direction.Up | Direction.Down | Direction.Left | Direction.Right
}
```

**Component Breakdown**
- `Direction` (without `typeof`): The enum type (union of members).

**Syntax Rules**

- `typeof EnumObject` produces the type of the enum object (the value).
- `EnumObject.Member` (e.g., `Direction.Up`) is a literal type.
- `EnumObject` (without `typeof`) is the enum type (union of member types).
- `keyof typeof EnumObject` gives the union of member names.
- `typeof EnumObject[keyof typeof EnumObject]` gives the union of member values.
- TypeScript 5.0+ makes all enums union enums, so `Direction` is a union of its members.
- Enum members are both values (at runtime) and types (at compile time).

**Constraints and Limitations**

- `typeof` on an enum gives the object type, not the enum type—using `typeof Direction` where `Direction` is expected is a common mistake.
- The enum object type includes both the forward mappings (name → value) and reverse mappings (value → name) for numeric enums, but not for string enums.
- `keyof typeof EnumObject` gives member names, not values.
- `typeof EnumObject[keyof typeof EnumObject]` gives the union of values.
- Enum member types are nominal, not structural—different enums with the same values are not interchangeable.

### Multiple Annotated Complete Step-by-Step Code Examples

#### Example 1: Enum Object vs. Enum Member Type

```typescript
// Step 1: Define a string enum.
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}

// Step 2: typeof Direction gives the enum object type.
type DirectionObject = typeof Direction;
// { Up: Direction.Up; Down: Direction.Down; Left: Direction.Left; Right: Direction.Right; }

// Step 3: Direction (without typeof) gives the enum type (union of members).
type DirectionType = Direction;
// Direction.Up | Direction.Down | Direction.Left | Direction.Right

// Step 4: Use the enum type in a function.
function move(direction: Direction): void {
  console.log(`Moving ${direction}`);
}

move(Direction.Up);     // "Moving UP"
move(Direction.Down);   // "Moving DOWN"

// Step 5: Direction.Up as a type.
type UpLiteral = Direction.Up;  // Direction.Up (literal type)

const up: UpLiteral = Direction.Up;  // ✅
// const wrong: UpLiteral = Direction.Down;  // ❌ Error.

// Step 6: keyof typeof gives member names.
type DirectionKey = keyof typeof Direction;
// "Up" | "Down" | "Left" | "Right"

function getDirectionName(key: DirectionKey): string {
  return key;
}

console.log(getDirectionName("Up"));  // "Up"

// Step 7: typeof Direction[keyof typeof Direction] gives member values.
type DirectionValue = typeof Direction[keyof typeof Direction];
// "UP" | "DOWN" | "LEFT" | "RIGHT"

function getDirectionValue(value: DirectionValue): void {
  console.log(`Value: ${value}`);
}

getDirectionValue("UP");  // "Value: UP"

// Step 8: Common pitfall — using typeof where the enum type is expected.
// function badMove(direction: typeof Direction): void { }
// ❌ Error: typeof Direction is the enum object, not the enum type.
```

**Expected Output:**
```
Moving UP
Moving DOWN
Up
Value: UP
```

**Why This Output Occurs:** `typeof Direction` gives the enum object type, which is not the same as the enum type `Direction`. The enum type is a union of member types. `keyof typeof Direction` gives member names, and `typeof Direction[keyof typeof Direction]` gives member values. Using `typeof Direction` as a type annotation is a common mistake—it refers to the object, not the union of members.

#### Example 2: Numeric Enum and Reverse Mappings

```typescript
// Step 1: Define a numeric enum.
enum Status {
  Pending,
  Active,
  Inactive,
}

// Step 2: Numeric enums have reverse mappings.
// Status[0] === "Pending", Status[1] === "Active", Status[2] === "Inactive"

// Step 3: typeof Status gives the enum object type.
type StatusObject = typeof Status;
// Includes forward mappings (Pending: Status.Pending) and reverse mappings (0: "Pending", etc.)

// Step 4: Status (without typeof) is the enum type.
type StatusType = Status;
// Status.Pending | Status.Active | Status.Inactive

function getStatusLabel(status: Status): string {
  switch (status) {
    case Status.Pending: return "Pending";
    case Status.Active: return "Active";
    case Status.Inactive: return "Inactive";
  }
}

console.log(getStatusLabel(Status.Pending));  // "Pending"

// Step 5: keyof typeof includes both forward and reverse mapping keys.
type StatusKey = keyof typeof Status;
// "Pending" | "Active" | "Inactive" (number keys are also present but not included in string union)

// Step 6: Accessing values through the enum object.
console.log(Status[0]);  // "Pending" (reverse mapping)
console.log(Status.Pending);  // 0 (forward mapping)

// Step 7: The enum type is nominal — different enums with the same values are not interchangeable.
enum OtherStatus {
  Pending,
  Active,
  Inactive,
}

// const wrong: Status = OtherStatus.Pending;  // ❌ Error: Type 'OtherStatus' is not assignable to type 'Status'.

console.log("Enum distinction demonstrated.");
```

**Expected Output:**
```
Pending
Pending
0
Enum distinction demonstrated.
```

**Why This Output Occurs:** `typeof Status` gives the enum object type, which includes both forward and reverse mappings for numeric enums. The enum type `Status` is a union of member types. `Status[0]` uses the reverse mapping to return `"Pending"`. Different enums with the same values are not interchangeable because enum types are nominal.

### Real-World Cases

**Case 1: API Status Codes**
APIs with numeric status codes use numeric enums, and `typeof Status[keyof typeof Status]` extracts the union of status code values for type-safe handling.

**Case 2: Form Field Names**
String enums for form field names use `keyof typeof FormField` to extract the union of field names for type-safe form handling.

**Case 3: Redux Action Types**
Redux action types defined as string enums use the enum type (not `typeof`) for type-safe action handling.

**Case 4: Configuration Keys**
Configuration keys defined as string enums use `keyof typeof ConfigKey` to extract the union of keys for type-safe configuration access.

**Case 5: State Machine States**
State machine states defined as string enums use the enum type for type-safe state transitions.

**Case 6: HTTP Methods**
HTTP methods defined as string enums use `typeof HttpMethod[keyof typeof HttpMethod]` to extract the union of method values for type-safe request handling.

---

## References

- TypeScript Handbook: Typeof Type Operator — https://www.typescriptlang.org/docs/handbook/2/typeof-types.html
- TypeScript Handbook: Indexed Access Types — https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
- TypeScript Handbook: Enums — https://www.typescriptlang.org/docs/handbook/enums.html
- TypeScript 5.0 Release Notes (Union Enums) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html
- TypeScript 4.9 Release Notes (`satisfies` Operator) — https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html
- Total TypeScript: Combine `keyof` and `typeof` to Derive Types — https://www.totaltypescript.com/workshops/typescript-pro-essentials/deriving-types-from-values/the-typeof-operator/solution
- Total TypeScript: The `typeof` Operator — https://www.totaltypescript.com/workshops/typescript-pro-essentials/deriving-types-from-values/the-typeof-operator
- TypeScript Playground: Typeof Type Operator — https://www.typescriptlang.org/play/typescript/primitives/typeof-types.ts.html
- Stack Overflow: Is there any way to check if an object is a type of enum in TypeScript? — https://stackoverflow.com/questions/42006725/is-there-any-way-to-check-if-an-object-is-a-type-of-enum-in-typescript
- Stack Overflow: `typeof` class in TypeScript — https://stackoverflow.com/questions/39392853/typeof-class-in-typescript
- MDN: `typeof` Operator — https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof
- TypeScript ESLint: no-unnecessary-type-assertion — https://typescript-eslint.io/rules/no-unnecessary-type-assertion/
- Effective TypeScript: Item 8 — Know How to Tell Whether a Symbol Is in the Type Space or Value Space