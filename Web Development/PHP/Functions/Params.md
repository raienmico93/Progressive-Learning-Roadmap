Here is a comprehensive research cheat sheet on PHP function parameters, structured for both academic clarity and beginner accessibility.

---

## Topic Overview: PHP Function Parameters

### Definitions

**Core Definition**
Function parameters are the named variables declared in a function's signature that define the inputs the function can accept. They allow data to be passed into a function when it is called.

**Technical Definition**
In PHP, function parameters are declared in the function signature within parentheses. The argument list is a comma-delimited list of expressions that are evaluated from left to right (eager evaluation) and assigned to the function's parameters before the function body executes. PHP supports passing arguments by value (the default), by reference, default argument values, variable-length argument lists (variadic parameters), and named arguments.

**Beginner-Friendly Explanation**
Think of a function as a recipe. The parameters are the ingredients the recipe asks for. Some recipes need specific ingredients (required parameters), some have suggested ingredients you can swap out (optional/default parameters), and some recipes let you add as many extra ingredients as you like (variadic parameters). Parameters give the recipe the flexibility to produce different results each time.

---

### Key Characteristics

- **Flexibility**: Parameters allow a function to operate on different data each time it is called.
- **Type Safety**: Type declarations enforce what kind of data can be passed, catching errors early.
- **Order Sensitivity**: Positional parameters are assigned in the order they are passed, though named arguments (PHP 8.0+) remove this restriction.
- **Value vs. Reference**: By default, arguments are passed by value (a copy); passing by reference (`&`) allows a function to modify the original variable.
- **Version-Specific Features**: Named arguments require PHP 8.0.0+; union types require PHP 8.0.0+; intersection types require PHP 8.1.0+; DNF types require PHP 8.2.0+.

---

### Prerequisites

- **PHP 8.0+ Recommended**: Named arguments and union types require PHP 8.0.0. Intersection types require PHP 8.1.0.
- **Basic PHP Syntax**: Understanding of variables, data types, and function declarations.
- **`declare(strict_types=1)` Awareness**: Strict typing changes how type declarations behave, enforcing exact type matches instead of coercing values.

---

### Related Programming Areas

- **Function Fundamentals**: Parameters are a core part of function declarations.
- **Type Systems**: PHP's gradual type system, including scalar, union, intersection, and nullable types.
- **Object-Oriented Programming**: Method signatures use the same parameter rules.
- **API Design**: Parameter design directly impacts the usability and readability of functions and methods.
- **Static Analysis**: Tools like PHPStan and Psalm rely on parameter type declarations to detect bugs.

---

## Core Concepts / Features

---

### 1. Positional Parameters

#### Definitions

**Core Definition**
Positional parameters are parameters that are assigned arguments based on the order in which they appear in both the function signature and the function call.

**Technical Definition**
In a positional parameter system, the first argument passed to a function is assigned to the first parameter, the second argument to the second parameter, and so on. Any missing argument is filled with its default value (if one exists), and superfluous arguments may be dropped or trigger an error depending on the function's variadic nature.

**Beginner-Friendly Explanation**
Positional parameters are like a queue: the first person in line gets the first ticket, the second person gets the second ticket. The order matters. If you call `greet("Alice", "Bob")`, the first parameter receives `"Alice"` and the second receives `"Bob"`.

#### Purposes

- To provide a simple, direct way to pass data to a function.
- To match the traditional calling convention used in most programming languages.
- To ensure predictable assignment of arguments when the parameter order is well-known.

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName($param1, $param2, $param3) {
    // body
}

// Calling with positional arguments
functionName($value1, $value2, $value3);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$param1, $param2` | Parameters declared in a fixed order. |
| `$value1, $value2` | Arguments passed in the corresponding order. |
| Comma separation | Parameters and arguments are separated by commas. |

**Syntax Rules**

- Arguments are evaluated from left to right and assigned to parameters in the same order.
- Missing arguments for parameters without defaults result in an `ArgumentCountError` (PHP 7.1+) or a warning (older versions).
- Extra arguments beyond the declared parameters are ignored unless the function uses variadic parameters.

**Constraints and Limitations**

- **Inflexible for optional parameters**: If a function has multiple optional parameters, you cannot skip one without providing all preceding optional parameters (unless using named arguments).
- **Order dependency**: Changing the parameter order in the function signature breaks all existing calls.

#### Annotated Code Examples

**Example 1: Basic Positional Parameters**

```php
<?php
// Step 1: Declare a function with three positional parameters
function describePet($species, $name, $age) {
    return "$name is a $age-year-old $species.\n";
}

// Step 2: Call with arguments in the exact order
echo describePet("Dog", "Rex", 5);
// Outputs: Rex is a 5-year-old Dog.
?>
```

**Expected Output:**
```
Rex is a 5-year-old Dog.
```

**Why This Output:**  
The arguments `"Dog"`, `"Rex"`, and `5` are assigned positionally to `$species`, `$name`, and `$age` respectively. The function concatenates them into a sentence.

**Step-by-Step Setup:**
1. Save as `positional.php`.
2. Run `php positional.php`.
3. The output confirms the order-based assignment.

---

**Example 2: Positional Parameters with Default Values**

```php
<?php
function makeCoffee($type = "Espresso", $size = "Medium") {
    return "Making a $size cup of $type.\n";
}

// Step 1: Call with no arguments — both defaults used
echo makeCoffee();
// Outputs: Making a Medium cup of Espresso.

// Step 2: Call with one argument — $type overridden, $size default
echo makeCoffee("Latte");
// Outputs: Making a Medium cup of Latte.

// Step 3: Call with two arguments — both overridden
echo makeCoffee("Cappuccino", "Large");
// Outputs: Making a Large cup of Cappuccino.
?>
```

**Expected Output:**
```
Making a Medium cup of Espresso.
Making a Medium cup of Latte.
Making a Large cup of Cappuccino.
```

**Why This Output:**  
Because parameters are positional, you must provide arguments in order. You cannot skip `$type` to set only `$size` without using named arguments (PHP 8.0+).

#### Real-World Cases

- **Mathematical Functions**: `pow($base, $exponent)` relies on positional order: the base comes first, the exponent second.
- **String Functions**: `str_replace($search, $replace, $subject)` expects arguments in a specific order; reversing them produces incorrect results.
- **Database Helpers**: `query($sql, $params)` expects the SQL string first and the parameter array second.

---

### 2. Default Parameters

#### Definitions

**Core Definition**
Default parameters allow a function to specify a fallback value for a parameter, which is used when the caller does not provide an argument for that parameter.

**Technical Definition**
A default value is declared using syntax similar to variable assignment: `$param = value`. The default is used only when the parameter's argument is not passed at all. Passing `null` explicitly does not trigger the default; `null` becomes the value. Default values may be scalar values, arrays, `null`, and as of PHP 8.1.0, objects created with `new ClassName()`.

**Beginner-Friendly Explanation**
A default parameter is like a backup plan. If you don't tell the function what coffee you want, it will make you a cappuccino by default. But if you explicitly say "nothing" (`null`), you get nothing—the backup plan is only used when you say nothing at all.

#### Purposes

- To make function calls simpler by allowing commonly used values to be omitted.
- To provide backward-compatible ways to add new parameters without breaking existing calls.
- To reduce the need for function overloading (which PHP does not support).

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName($requiredParam, $optionalParam = defaultValue) {
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$optionalParam = defaultValue` | Assigns a default value to an optional parameter. |
| `defaultValue` | Must be a constant expression (scalar, array, `null`, or as of PHP 8.1.0, an object via `new`). |

**Syntax Rules**

- Optional parameters should be declared **after** required parameters. Declaring a required parameter after an optional one is deprecated as of PHP 8.0.0.
- The default is used only when the argument is **omitted entirely**, not when `null` is passed.
- As of PHP 8.0.0, named arguments can be used to skip over multiple optional parameters regardless of order.

**Constraints and Limitations**

- **Constant expressions only**: The default value cannot be a variable, class member, or function call (prior to PHP 8.1.0 for objects).
- **Null ambiguity**: Passing `null` to a parameter with a non-nullable default will assign `null` (and may cause a type error if the parameter is typed and strict types are enabled). To make a parameter explicitly nullable with a `null` default, use a nullable type declaration (e.g., `?string $name = null`), though this implicit-nullable form is deprecated as of PHP 8.4.0.

#### Annotated Code Examples

**Example 1: Default Parameter Values**

```php
<?php
function makeCoffee($type = "cappuccino") {
    return "Making a cup of $type.\n";
}

// Step 1: No argument — default used
echo makeCoffee();
// Outputs: Making a cup of cappuccino.

// Step 2: Passing null — null is used, NOT the default
echo makeCoffee(null);
// Outputs: Making a cup of .

// Step 3: Argument provided — overrides default
echo makeCoffee("espresso");
// Outputs: Making a cup of espresso.
?>
```

**Expected Output:**
```
Making a cup of cappuccino.
Making a cup of .
Making a cup of espresso.
```

**Why This Output:**  
When `makeCoffee()` is called without an argument, the default `"cappuccino"` is used. When `null` is passed, `$type` becomes `null`, and the string interpolation produces an empty string. Only when an actual argument is passed is the default overridden.

---

**Example 2: Default Array Parameter**

```php
<?php
function makeCoffee($types = array("cappuccino"), $coffeeMaker = null) {
    $device = is_null($coffeeMaker) ? "hands" : $coffeeMaker;
    return "Making a cup of " . join(", ", $types) . " with $device.\n";
}

echo makeCoffee();
// Outputs: Making a cup of cappuccino with hands.

echo makeCoffee(array("cappuccino", "lavazza"), "teapot");
// Outputs: Making a cup of cappuccino, lavazza with teapot.
?>
```

**Expected Output:**
```
Making a cup of cappuccino with hands.
Making a cup of cappuccino, lavazza with teapot.
```

**Why This Output:**  
Arrays and `null` are valid default values. When the second argument is omitted, `$coffeeMaker` is `null`, and the ternary operator selects `"hands"` as the device.

#### Real-World Cases

- **Configuration Functions**: `connectDatabase($host = "localhost", $port = 3306)` uses sensible defaults for local development while allowing overrides in production.
- **Logging**: `logMessage($message, $level = "INFO")` allows most calls to omit the level, using a default of `"INFO"`.
- **HTML Helpers**: `htmlspecialchars($string, $flags = ENT_QUOTES | ENT_SUBSTITUTE, $encoding = "UTF-8")` provides default flags and encoding.

---

### 3. Optional Parameters

#### Definitions

**Core Definition**
Optional parameters are parameters that do not require an argument to be passed when the function is called. They are created by assigning a default value in the function signature.

**Technical Definition**
In PHP, a parameter becomes optional when it has a default value. If no argument is supplied for that parameter, the default value is used. Optional parameters must generally be declared after required parameters (though named arguments relax this ordering constraint for calls). As of PHP 8.0.0, declaring a required parameter after an optional one is deprecated.

**Beginner-Friendly Explanation**
Optional parameters are "nice-to-have" inputs. The function will still work if you don't provide them—it will just use a pre-set value. They are the opposite of required parameters, which will cause an error if omitted.

#### Purposes

- To reduce the number of arguments callers must remember and provide.
- To make functions easier to use in common cases while retaining flexibility for advanced cases.
- To support API evolution by adding new optional features without breaking existing calls.

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName($required, $optional = defaultValue) {
    // body
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `$optional = defaultValue` | Declares an optional parameter with a fallback value. |
| Position | Must come after all required parameters (unless using named arguments to call). |

**Syntax Rules**

- A parameter is optional if and only if it has a default value.
- Optional parameters must follow required parameters in the signature; otherwise, the default may never be reachable, and PHP 8.0+ emits a deprecation notice.
- Named arguments (PHP 8.0+) allow skipping optional parameters regardless of their position in the signature.

**Constraints and Limitations**

- **Ordering**: Traditional positional calls cannot skip an optional parameter to reach a later one.
- **`null` handling**: An optional parameter with a non-nullable default will still accept `null` if passed, which may lead to unexpected behavior or type errors under strict typing.

#### Annotated Code Examples

**Example 1: Incorrect vs. Correct Ordering**

```php
<?php
// INCORRECT: Optional parameter before required parameter
// function makeYogurt($container = "bowl", $flavour) { ... }
// Calling makeYogurt("raspberry") assigns "raspberry" to $container
// and leaves $flavour missing — deprecated and error-prone.

// CORRECT: Required parameter first
function makeYogurt($flavour, $container = "bowl") {
    return "Making a $container of $flavour yogurt.\n";
}

echo makeYogurt("raspberry");
// Outputs: Making a bowl of raspberry yogurt.
?>
```

**Expected Output:**
```
Making a bowl of raspberry yogurt.
```

**Why This Output:**  
With the required parameter first, the argument `"raspberry"` is correctly assigned to `$flavour`, and the optional `$container` uses its default `"bowl"`.

---

**Example 2: Skipping Optional Parameters with Named Arguments (PHP 8.0+)**

```php
<?php
function makeYogurt($container = "bowl", $flavour = "raspberry", $style = "Greek") {
    return "Making a $container of $flavour $style yogurt.\n";
}

// Skip $container and $flavour, set only $style
echo makeYogurt(style: "natural");
// Outputs: Making a bowl of raspberry natural yogurt.
?>
```

**Expected Output:**
```
Making a bowl of raspberry natural yogurt.
```

**Why This Output:**  
Named arguments allow the caller to target `$style` directly without providing values for the preceding optional parameters `$container` and `$flavour`.

#### Real-World Cases

- **String Formatting**: `number_format($num, $decimals = 0, $decimal_separator = ".", $thousands_separator = ",")` allows callers to use defaults for common formatting.
- **Array Functions**: `array_slice($array, $offset, $length = null, $preserve_keys = false)` makes the length and key-preservation behavior optional.
- **HTTP Helpers**: `redirect($url, $statusCode = 302)` uses a default redirect status.

---

### 4. Variadic Parameters (`...$args`)

#### Definitions

**Core Definition**
Variadic parameters allow a function to accept a variable number of arguments, which are collected into a single array parameter.

**Technical Definition**
A variadic parameter is declared by prefixing a parameter name with the `...` token (e.g., `...$numbers`). All remaining arguments passed to the function that do not match preceding positional parameters are collected into the variadic parameter as an array. The `...` token can also be used at call time to unpack an array or `Traversable` into an argument list.

**Beginner-Friendly Explanation**
Variadic parameters are like a "catch-all" basket. If a function has a variadic parameter, you can throw as many extra arguments into it as you want, and the function will receive them all as a neat array. It is perfect for functions like `sum()` that need to add up any number of values.

#### Purposes

- To allow functions to accept an unlimited number of arguments without requiring the caller to pass an array explicitly.
- To replace older patterns like `func_get_args()` with cleaner, type-safe syntax.
- To enable functions like `printf`-style formatters or mathematical aggregators.

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName($fixedParam, ...$variadicParam) {
    // $variadicParam is an array
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `...$variadicParam` | The `...` token followed by a parameter name. Collects all remaining arguments into an array. |
| Position | Must be the **last** parameter in the signature. |
| Type declaration | Optional type can precede `...` (e.g., `DateInterval ...$intervals`); all collected arguments must match that type. |

**Syntax Rules**

- The variadic parameter must be the last parameter declared.
- Arguments captured by `...` are passed as an array, even if only one argument is provided.
- The `...` token can also be used in function calls to unpack an array: `foo(...$array)`.
- As of PHP 8.0.0, named arguments that do not match any declared parameter are collected into the variadic parameter's array (with string keys).

**Constraints and Limitations**

- **Type enforcement**: If a type is declared before `...`, every captured argument must satisfy that type, or a `TypeError` is thrown.
- **Only one variadic**: A function can have only one variadic parameter, and it must be last.
- **By-reference variadics**: Variadic parameters can be passed by reference by prefixing `&` before `...` (e.g., `&...$args`), but this is rarely used.

#### Annotated Code Examples

**Example 1: Summing Variable Arguments**

```php
<?php
function sum(...$numbers) {
    $acc = 0;
    foreach ($numbers as $n) {
        $acc += $n;
    }
    return $acc;
}

echo sum(1, 2, 3, 4);
// Outputs: 10
?>
```

**Expected Output:**
```
10
```

**Why This Output:**  
The `...$numbers` parameter collects `1`, `2`, `3`, and `4` into the array `[1, 2, 3, 4]`. The `foreach` loop sums the elements to produce `10`.

---

**Example 2: Unpacking an Array at Call Time**

```php
<?php
function add($a, $b) {
    return $a + $b;
}

echo add(...[1, 2]) . "\n"; // Outputs: 3

$args = [3, 4];
echo add(...$args) . "\n";  // Outputs: 7
?>
```

**Expected Output:**
```
3
7
```

**Why This Output:**  
The `...` operator at the call site spreads the array elements into separate arguments. `[1, 2]` becomes `add(1, 2)`, and `[3, 4]` becomes `add(3, 4)`.

---

**Example 3: Typed Variadic Parameters**

```php
<?php
function totalIntervals($unit, DateInterval ...$intervals) {
    $time = 0;
    foreach ($intervals as $interval) {
        $time += $interval->$unit;
    }
    return $time;
}

$a = new DateInterval('P1D');
$b = new DateInterval('P2D');
echo totalIntervals('d', $a, $b) . ' days';
// Outputs: 3 days
?>
```

**Expected Output:**
```
3 days
```

**Why This Output:**  
The type declaration `DateInterval` before `...` ensures that every captured argument is a `DateInterval` object. Passing a non-`DateInterval` (e.g., `null`) would throw a `TypeError`.

#### Real-World Cases

- **Mathematical Utilities**: `sum(...$numbers)`, `average(...$scores)`, `max(...$values)`.
- **Logging**: `log($message, ...$context)` collects additional context data into an array.
- **Middleware Pipelines**: `pipe($input, ...$callbacks)` passes any number of transformation functions.
- **SQL Query Builders**: `where(...$conditions)` accepts multiple condition fragments.

---

### 5. Named Arguments (PHP 8.0+)

#### Definitions

**Core Definition**
Named arguments allow a caller to pass an argument to a function by specifying the parameter's name, rather than relying on its position.

**Technical Definition**
Named arguments are passed by prefixing the value with the parameter name followed by a colon (e.g., `paramName: $value`). They were introduced in PHP 8.0.0 as an extension of positional parameters. Named arguments make the meaning of the argument self-documenting, make arguments order-independent, and allow skipping default values arbitrarily. The parameter name must be an identifier; dynamic parameter names are not allowed.

**Beginner-Friendly Explanation**
Named arguments are like addressing an envelope: instead of just writing the contents and hoping they go to the right person, you write "To: Alice" on the envelope. You specify exactly which parameter gets which value, so you don't have to worry about the order.

#### Purposes

- To make function calls self-documenting by clarifying what each argument represents.
- To allow skipping optional parameters without providing values for all preceding optional parameters.
- To make calls order-independent, improving readability for functions with many parameters.
- To reduce bugs caused by positional mistakes when functions have several same-typed parameters.

#### Syntax Rules and Structure

**General Syntax**
```php
functionName(paramName: $value, anotherParam: $anotherValue);
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `paramName` | The exact name of the parameter as declared in the function signature (without the `$`). Case-sensitive. |
| `:` | Separator between the parameter name and the value. |
| `$value` | The argument value. |

**Syntax Rules**

- Named arguments must come **after** any positional arguments: `foo($pos1, named: $value)` is valid; `foo(named: $value, $pos1)` is not.
- Named and positional arguments can be mixed, but positional arguments must come first.
- Passing an argument to the same named parameter multiple times throws an `Error`.
- As of PHP 8.1.0, named arguments can be used after unpacking, but a named argument must not override an already unpacked argument.
- Reserved keywords are allowed as parameter names in named arguments.

**Constraints and Limitations**

- **Dynamic names not allowed**: The parameter name must be a literal identifier; you cannot use a variable as the parameter name (e.g., `$name: $value` is invalid).
- **Case-sensitive**: Named arguments are case-sensitive, unlike function names.
- **PHP 8.0.0+ required**: This feature is not available in earlier versions.

#### Annotated Code Examples

**Example 1: Basic Named Arguments**

```php
<?php
function createUser($username, $email, $role = "member") {
    return "User: $username, Email: $email, Role: $role\n";
}

// Step 1: Use named arguments to skip $role
echo createUser(username: "alice", email: "alice@example.com");
// Outputs: User: alice, Email: alice@example.com, Role: member

// Step 2: Named arguments can be in any order
echo createUser(email: "bob@example.com", username: "bob", role: "admin");
// Outputs: User: bob, Email: bob@example.com, Role: admin
?>
```

**Expected Output:**
```
User: alice, Email: alice@example.com, Role: member
User: bob, Email: bob@example.com, Role: admin
```

**Why This Output:**  
Named arguments allow the caller to target specific parameters by name. In the first call, `$role` is omitted and its default `"member"` is used. In the second call, the order of named arguments is irrelevant because each value is mapped by name.

---

**Example 2: Combining Positional and Named Arguments**

```php
<?php
function htmlspecialcharsWrapper($string, $flags = ENT_QUOTES, $encoding = "UTF-8", $doubleEncode = true) {
    return htmlspecialchars($string, $flags, $encoding, $doubleEncode);
}

// Positional $string, named $doubleEncode
$result = htmlspecialcharsWrapper("<p>Hello</p>", doubleEncode: false);
echo $result . "\n";
// Outputs: &lt;p&gt;Hello&lt;/p&gt;
?>
```

**Expected Output:**
```
&lt;p&gt;Hello&lt;/p&gt;
```

**Why This Output:**  
The first argument `"<p>Hello</p>"` is passed positionally to `$string`. The named argument `doubleEncode: false` targets the fourth parameter directly, skipping `$flags` and `$encoding`, which use their defaults.

---

**Example 3: Error — Duplicate Named Argument**

```php
<?php
function foo($param) {
    return $param;
}

try {
    foo(param: 1, param: 2);
} catch (Error $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
// Outputs: Error: Named parameter $param overwrites previous argument
?>
```

**Expected Output:**
```
Error: Named parameter $param overwrites previous argument
```

**Why This Output:**  
Passing the same named parameter twice is an error. PHP throws an `Error` exception because the second value would overwrite the first, indicating a likely mistake.

#### Real-World Cases

- **API Wrappers**: `httpRequest(url: $url, method: "POST", timeout: 30)` makes the call self-documenting.
- **Configuration Objects**: `new Config(cacheEnabled: true, logLevel: "debug")` clarifies which boolean flags are being set.
- **Skipping Optional Parameters**: `setcookie(name: "session", value: $id, httponly: true)` targets only the relevant options without filling in all preceding defaults.

---

### 6. Type Declarations

#### Definitions

**Core Definition**
Type declarations specify the expected data type of a parameter (and optionally a return value), ensuring that only values of the correct type are accepted.

**Technical Definition**
Type declarations can be added to function arguments, return values, and (as of PHP 7.4.0) class properties, and (as of PHP 8.3.0) class constants. If the value passed does not match the declared type, a `TypeError` is thrown. PHP supports a wide range of types, including scalar types (`int`, `float`, `string`, `bool`), `array`, `object`, `iterable`, `callable`, class/interface types, `mixed`, `void`, `never`, `null`, `false`, `true`, and composite types (nullable, union, intersection).

**Beginner-Friendly Explanation**
Type declarations are like specifying the shape of a puzzle piece. If a function expects an `int`, you cannot shove a `string` into it—PHP will complain. This helps catch mistakes early, before they cause confusing errors deeper in your code.

#### Purposes

- To catch type-related bugs at the call site, rather than deep inside a function.
- To make function signatures self-documenting and easier to understand.
- To enable static analysis tools (PHPStan, Psalm) to detect issues before runtime.
- To enforce data integrity in APIs and domain logic.

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName(Type $param): ReturnType {
    // body
}
```

**Component Breakdown**

| Type Category | Examples | Availability |
|--------------|----------|--------------|
| Scalar | `int`, `float`, `string`, `bool` | PHP 7.0+ |
| Compound | `array`, `object`, `iterable`, `callable` | Varies |
| Class/Interface | `User`, `Countable` | PHP 5.0+ |
| Nullable | `?string` (sugar for `string|null`) | PHP 7.1+ |
| Union | `int|string` | PHP 8.0+ |
| Intersection | `Countable&Traversable` | PHP 8.1+ |
| DNF | `(A&B)|null` | PHP 8.2+ |
| Literal | `true`, `false`, `null` (standalone) | PHP 8.2+ |

**Syntax Rules**

- A type declaration precedes the parameter variable: `int $age`.
- Return types follow a colon after the parameter list: `function foo(): int`.
- With `declare(strict_types=1)`, scalar type declarations enforce exact type matching and do not coerce.
- Without strict types, PHP may coerce scalar values (e.g., `"5"` to `5` for an `int` parameter).

**Constraints and Limitations**

- **`resource` cannot be used** in user-land type declarations (though it can be used in PHPDoc).
- **`void` and `never`** are return-only types and cannot be used for parameters.
- **Variance rules**: When overriding a method or implementing an interface, parameter types must be contravariant (wider or equal), and return types covariant (narrower or equal).

#### Annotated Code Examples

**Example 1: Basic Type Declarations**

```php
<?php
declare(strict_types=1);

function addNumbers(int $a, int $b): int {
    return $a + $b;
}

echo addNumbers(5, 10) . "\n"; // Outputs: 15

// Passing a string throws a TypeError under strict types
try {
    echo addNumbers("5", 10);
} catch (TypeError $e) {
    echo "Type error: " . $e->getMessage() . "\n";
}
?>
```

**Expected Output:**
```
15
Type error: addNumbers(): Argument #1 ($a) must be of type int, string given
```

**Why This Output:**  
With `strict_types=1`, the `int` type declarations enforce exact matching. Passing `"5"` (a string) instead of `5` (an integer) throws a `TypeError`.

---

**Example 2: Class Type Declaration**

```php
<?php
class User {
    public function __construct(public string $name) {}
}

function greetUser(User $user): string {
    return "Hello, " . $user->name . "!\n";
}

$user = new User("Alice");
echo greetUser($user);
// Outputs: Hello, Alice!
?>
```

**Expected Output:**
```
Hello, Alice!
```

**Why This Output:**  
The parameter is declared as `User $user`, so only `User` objects (or subclasses) are accepted. The function accesses the public `$name` property and returns a greeting.

#### Real-World Cases

- **Domain-Driven Design**: Value objects (e.g., `EmailAddress`, `Money`) are passed as typed parameters, preventing primitive obsession.
- **Repository Interfaces**: `findById(int $id): ?User` uses an `int` parameter and a nullable `User` return type.
- **Service Layers**: `charge(PaymentMethod $method, float $amount): PaymentResult` enforces that only valid payment method objects are passed.

---

### 7. Nullable Parameters

#### Definitions

**Core Definition**
A nullable parameter is a parameter that explicitly accepts either a value of a specified type or `null`.

**Technical Definition**
Nullable types were introduced in PHP 7.1.0. A nullable type is declared by prefixing the type with a question mark: `?Type`. This is syntactic sugar for `Type|null`. For multi-type nullable declarations, the full union syntax (`int|string|null`) is recommended, especially since the implicit-nullable form (`Type $param = null`) is deprecated as of PHP 8.4.0.

**Beginner-Friendly Explanation**
A nullable parameter is a parameter that says, "I'll accept a value of this type, or I'll accept nothing at all (`null`)." It's like a form field that is optional: you can fill it in with a valid value, or you can leave it blank.

#### Purposes

- To explicitly allow `null` as a valid input when a value may be absent.
- To make the possibility of `null` visible in the function signature, improving code clarity.
- To support optional dependencies or lookup results that may not exist.

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName(?Type $param) {
    // $param is Type or null
}
```

For multi-type nullables:
```php
function functionName(int|string|null $param) {
    // ...
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `?Type` | Nullable type sugar for `Type|null`. |
| `Type|null` | Full union syntax; recommended for multi-type nullables. |

**Syntax Rules**

- The `?` prefix applies to a single type: `?int` means `int|null`.
- For union types, include `null` as a member: `int|string|null`.
- `null` must be the last member in a union declaration (though this is a convention, not a strict parser rule).

**Constraints and Limitations**

- **Implicit nullable deprecation**: Declaring `Type $param = null` (without `?`) makes the type implicitly nullable, but this is deprecated as of PHP 8.4.0. Use an explicit nullable type instead.
- **Standalone `null` type**: As of PHP 8.2.0, `null` can be used as a standalone type.

#### Annotated Code Examples

**Example 1: Nullable Parameter with `?Type`**

```php
<?php
declare(strict_types=1);

function greet(?string $name): string {
    if ($name === null) {
        return "Hello, guest!\n";
    }
    return "Hello, $name!\n";
}

echo greet("Alice"); // Outputs: Hello, Alice!
echo greet(null);    // Outputs: Hello, guest!
?>
```

**Expected Output:**
```
Hello, Alice!
Hello, guest!
```

**Why This Output:**  
The `?string` type accepts either a `string` or `null`. When `null` is passed, the function checks for it and returns a generic greeting.

---

**Example 2: Multi-Type Nullable with Union**

```php
<?php
declare(strict_types=1);

function formatValue(int|string|null $value): string {
    if ($value === null) {
        return "No value";
    }
    return "Value: $value";
}

echo formatValue(42) . "\n";       // Outputs: Value: 42
echo formatValue("hello") . "\n";  // Outputs: Value: hello
echo formatValue(null) . "\n";     // Outputs: No value
?>
```

**Expected Output:**
```
Value: 42
Value: hello
No value
```

**Why This Output:**  
The union type `int|string|null` explicitly allows integers, strings, and `null`. The function handles each case accordingly.

#### Real-World Cases

- **Optional Relationships**: `findUser(?int $id)` might return no user if `$id` is `null`.
- **Configuration**: `setTimeout(?int $seconds = null)` where `null` means "use the system default."
- **Data Mapping**: `mapValue(string $key, mixed $default = null)` uses a nullable `mixed` default to signal that no default was provided.

---

### 8. Union Types (PHP 8.0+)

#### Definitions

**Core Definition**
A union type allows a parameter to accept values of two or more different types.

**Technical Definition**
Union types were introduced in PHP 8.0.0. They are declared by joining types with the pipe (`|`) operator: `int|string`, `Foo|Bar|null`. A union type means the value must be **one of** the listed types. Union types can include scalar types, class types, `null`, `false`, and `true` (with version-specific rules).

**Beginner-Friendly Explanation**
A union type is like a multiple-choice question: "This parameter can be an integer **or** a string." You can pass either one, and PHP will accept it as long as it matches one of the allowed types.

#### Purposes

- To accept different types of input for the same logical operation (e.g., an ID that can be an integer or a string).
- To provide flexibility while maintaining type safety.
- To replace loose, untyped parameters with a documented set of acceptable types.

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName(int|string $param) {
    // $param is either int or string
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `TypeA|TypeB` | Union of types. The value must match at least one. |
| `null` in union | Allowed; `null` must be last in the union. |
| Duplicate types | Not allowed (e.g., `int|int` is a compile-time error). |

**Syntax Rules**

- Union types require PHP 8.0.0 or later.
- `void` cannot be part of a union.
- `false` and `true` can be used in unions; as of PHP 8.2.0, they can also be used as standalone types.
- DNF types (PHP 8.2+) allow combining union and intersection types in parentheses: `(A&B)|null`.

**Constraints and Limitations**

- **Redundancy**: Including a type that is a subtype of another in the same union (e.g., `object|User`) is redundant but may trigger a compile-time warning in some cases.
- **No `void`**: `void` is a return-only type and cannot be used in a union.
- **Coercion**: Without `strict_types=1`, scalar union members may still coerce values (e.g., a numeric string may be accepted for an `int` member).

#### Annotated Code Examples

**Example 1: Basic Union Type**

```php
<?php
declare(strict_types=1);

function formatId(int|string $id): string {
    return "ID: $id\n";
}

echo formatId(42);       // Outputs: ID: 42
echo formatId("abc-123"); // Outputs: ID: abc-123
?>
```

**Expected Output:**
```
ID: 42
ID: abc-123
```

**Why This Output:**  
The parameter `$id` is declared as `int|string`, so it accepts both integers and strings. The function simply concatenates the value into a string.

---

**Example 2: Union with `null`**

```php
<?php
declare(strict_types=1);

function getDiscount(int|float|null $amount): float {
    if ($amount === null) {
        return 0.0;
    }
    return $amount * 0.1;
}

echo getDiscount(100) . "\n";   // Outputs: 10
echo getDiscount(99.5) . "\n";  // Outputs: 9.95
echo getDiscount(null) . "\n";  // Outputs: 0
?>
```

**Expected Output:**
```
10
9.95
0
```

**Why This Output:**  
The union `int|float|null` accepts integers, floats, and `null`. The function checks for `null` explicitly and computes a discount for numeric values.

#### Real-World Cases

- **Identifiers**: `findUser(int|string $id)` allows looking up a user by numeric ID or string UUID.
- **Configuration Values**: `setOption(string $key, int|string|bool $value)` accepts different scalar configuration values.
- **API Responses**: `jsonResponse(array|object $data)` accepts either arrays or objects for serialization.

---

### 9. Intersection Types (PHP 8.1+)

#### Definitions

**Core Definition**
An intersection type requires a value to satisfy **all** of the listed types (interfaces or classes) simultaneously.

**Technical Definition**
Intersection types were introduced in PHP 8.1.0. Individual types forming the intersection are joined by the ampersand (`&`) symbol: `T&U&V` means the value must be an instance of `T` **and** `U` **and** `V`. Intersection types can only contain class and interface types (no scalar types, `self`, `parent`, or `static`).

**Beginner-Friendly Explanation**
An intersection type is like a job requirement: "The candidate must know PHP **and** SQL **and** JavaScript." The value must satisfy every requirement simultaneously. For example, a parameter typed as `Countable&Traversable` must be an object that implements both the `Countable` and `Traversable` interfaces.

#### Purposes

- To require an object to implement multiple interfaces at once.
- To express complex type constraints that cannot be captured by a single interface.
- To improve type safety in APIs that depend on multiple capabilities of an object.

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName(InterfaceA&InterfaceB $param) {
    // $param must implement both InterfaceA and InterfaceB
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `TypeA&TypeB` | Intersection of class/interface types. |
| Parentheses (DNF) | As of PHP 8.2.0, intersections can be combined with unions using parentheses: `(A&B)|null`. |

**Syntax Rules**

- Intersection types require PHP 8.1.0 or later.
- Only class and interface types are allowed; scalars, `self`, `parent`, and `static` are not permitted.
- Redundant intersections (e.g., `A&A`) are not allowed.
- Prior to PHP 8.2.0, intersection types could not be combined with union types.

**Constraints and Limitations**

- **Class-only**: You cannot write `int&string` because scalar types are not permitted in intersections.
- **No `void`**: Return-only types cannot be used in intersections.
- **DNF complexity**: Combining intersections and unions requires DNF syntax, which can become complex and should be used judiciously.

#### Annotated Code Examples

**Example 1: Intersection of Two Interfaces**

```php
<?php
declare(strict_types=1);

interface Loggable {
    public function log(): string;
}

interface Cacheable {
    public function cacheKey(): string;
}

class UserActivity implements Loggable, Cacheable {
    public function log(): string {
        return "User logged in";
    }
    public function cacheKey(): string {
        return "activity_123";
    }
}

function process(Loggable&Cacheable $item): void {
    echo $item->log() . " | Cache key: " . $item->cacheKey() . "\n";
}

$activity = new UserActivity();
process($activity);
// Outputs: User logged in | Cache key: activity_123
?>
```

**Expected Output:**
```
User logged in | Cache key: activity_123
```

**Why This Output:**  
The parameter `$item` is typed as `Loggable&Cacheable`, so it must implement both interfaces. `UserActivity` does, so it is accepted, and both methods can be safely called.

---

**Example 2: DNF Type (PHP 8.2+)**

```php
<?php
declare(strict_types=1);

interface A { public function a(): string; }
interface B { public function b(): string; }

class AB implements A, B {
    public function a(): string { return "A"; }
    public function b(): string { return "B"; }
}

function handle((A&B)|null $obj): string {
    if ($obj === null) {
        return "No object";
    }
    return $obj->a() . $obj->b();
}

echo handle(new AB()) . "\n"; // Outputs: AB
echo handle(null) . "\n";     // Outputs: No object
?>
```

**Expected Output:**
```
AB
No object
```

**Why This Output:**  
The DNF type `(A&B)|null` means the value can be either an object implementing both `A` and `B`, or `null`. This combines intersection and union types.

#### Real-World Cases

- **Collection Pipelines**: A parameter typed as `Countable&Traversable` ensures the value can be counted and iterated with `foreach`.
- **Caching Layers**: `Cacheable&Loggable $item` ensures an object can both be cached and logged.
- **Event Systems**: `StoppableEvent&PsrEvent $event` requires an event object to satisfy multiple PSR interfaces.

---

### 10. Reference Parameters (`&$param`)

#### Definitions

**Core Definition**
A reference parameter allows a function to modify the original variable passed by the caller, rather than working on a copy.

**Technical Definition**
By default, PHP passes arguments by value, meaning the function receives a copy and any modifications do not affect the original variable outside the function. To pass by reference, an ampersand (`&`) is prepended to the parameter name in the function definition: `function foo(&$param)`. This allows the function to modify the original variable. It is an error to pass a constant expression (e.g., `5 + 3`) to a reference parameter.

**Beginner-Friendly Explanation**
Passing by reference is like giving someone the actual key to your house instead of a copy. If they change the locks, your house is affected. Normally, PHP gives functions a copy of the key (pass by value), so changes don't affect you. Using `&` gives the function direct access to your original variable.

#### Purposes

- To allow a function to modify its arguments (e.g., sorting an array in place).
- To avoid the memory and performance cost of copying large data structures (though PHP's copy-on-write already mitigates this).
- To return multiple values from a function by modifying parameters passed by reference (a C-style output parameter pattern).

#### Syntax Rules and Structure

**General Syntax**
```php
function functionName(&$param) {
    // modifications to $param affect the caller's variable
}
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `&$param` | The `&` symbol before the parameter name declares it as passed by reference. |
| Call site | No `&` is needed at the call site; the reference is determined by the function signature. |

**Syntax Rules**

- The `&` must appear in the **function definition**, not at the call site (call-time pass-by-reference was deprecated in PHP 5.3 and removed).
- Reference parameters may have default values.
- Variadic parameters can be passed by reference using `&...$args`.

**Constraints and Limitations**

- **Constant expressions not allowed**: You cannot pass a literal or expression to a reference parameter; only variables (or other referenceable values) can be passed.
- **Objects**: Since PHP 5, objects are passed by handle (effectively by reference) for the purpose of method calls and property access, but the variable itself is still passed by value. To modify the variable to point to a different object, a reference parameter is still needed.
- **Performance**: PHP uses copy-on-write, so passing large arrays by value is often as fast as by reference unless the array is modified inside the function. Using `&` unnecessarily can make code harder to reason about.

#### Annotated Code Examples

**Example 1: Modifying a String by Reference**

```php
<?php
function addSomeExtra(&$string) {
    $string .= 'and something extra.';
}

$str = 'This is a string, ';
addSomeExtra($str);
echo $str . "\n";
// Outputs: This is a string, and something extra.
?>
```

**Expected Output:**
```
This is a string, and something extra.
```

**Why This Output:**  
The `&` before `$string` makes it a reference parameter. When the function appends to `$string`, it modifies the original `$str` variable outside the function.

---

**Example 2: Error — Passing a Constant Expression**

```php
<?php
function doubleValue(&$num) {
    $num *= 2;
}

try {
    doubleValue(5); // Constant expression, not a variable
} catch (Error $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
// Outputs: Error: doubleValue(): Argument #1 ($num) cannot be passed by reference
?>
```

**Expected Output:**
```
Error: doubleValue(): Argument #1 ($num) cannot be passed by reference
```

**Why This Output:**  
Reference parameters require a variable because the function needs a memory location to modify. A literal `5` has no variable location, so PHP throws an error.

---

**Example 3: Multiple Return Values via References**

```php
<?php
function divideWithRemainder(int $a, int $b, &$remainder): int {
    $remainder = $a % $b;
    return intdiv($a, $b);
}

$rem = 0;
$quotient = divideWithRemainder(17, 5, $rem);
echo "Quotient: $quotient, Remainder: $rem\n";
// Outputs: Quotient: 3, Remainder: 2
?>
```

**Expected Output:**
```
Quotient: 3, Remainder: 2
```

**Why This Output:**  
The function returns the quotient normally, while the remainder is written to the `$remainder` reference parameter, which is the original `$rem` variable. This pattern effectively returns two values from one function.

#### Real-World Cases

- **Built-in Functions**: `sort(array &$array)` sorts an array in place; `preg_match($pattern, $subject, array &$matches)` fills the `$matches` array by reference.
- **Output Parameters**: `exec($command, array &$output, int &$result_code)` uses reference parameters to return multiple pieces of information.
- **In-Place Mutation**: `array_push(array &$array, mixed ...$values)` modifies the original array.

---

### 11. Parameter Best Practices

#### Definitions

**Core Definition**
Parameter best practices are community-established guidelines for designing function signatures that are readable, safe, and maintainable.

**Technical Definition**
Best practices for PHP parameters include: using native type declarations on every parameter; declaring `strict_types=1` in every file; preferring default values for optional parameters; keeping parameter lists short; using named arguments for boolean flags and optional parameters; avoiding reference parameters unless necessary; and following PSR-12 coding standards.

**Beginner-Friendly Explanation**
Best practices are like the rules of good cooking: label your ingredients clearly (type declarations), don't overload the pan (keep parameter lists short), and follow the recipe consistently (PSR-12). Following them makes your code easier for you and others to use and maintain.

#### Purposes

- To catch type errors at the call site rather than deep inside a function.
- To make function signatures self-documenting and easy to understand.
- To improve compatibility with static analysis tools and IDEs.
- To reduce bugs caused by positional mistakes, ambiguous parameters, and type coercion.

#### Syntax Rules and Structure

**General Guidelines**

| Practice | Description |
|----------|-------------|
| **Always declare parameter types** | Every parameter should have a native type declaration. Use PHPDoc `@param` only for constraints native types cannot express (e.g., generic array shapes). |
| **Use `declare(strict_types=1)`** | This enforces exact type matching and prevents silent coercion. It should be the first statement in the file. |
| **Keep parameter lists short** | Prefer 0–3 parameters. For more, consider parameter objects or arrays. |
| **Use named arguments for clarity** | Use named arguments for boolean flags and optional parameters where the value alone does not convey meaning (e.g., `true`, `0`, `null`). |
| **Avoid unnecessary references** | Use `&` only when the function must modify the caller's variable. PHP's copy-on-write makes pass-by-value efficient for large arrays that are not modified. |
| **Place optional parameters last** | Required parameters should precede optional ones. Declaring a required parameter after an optional one is deprecated as of PHP 8.0.0. |
| **Follow PSR-12** | Use `camelCase` for function names, consistent indentation, and one parameter per line for long signatures. |

**Syntax Rules**

- `declare(strict_types=1);` must be the very first statement in the file (after the opening `<?php` tag).
- Type declarations should be as specific as possible (e.g., `int` rather than `mixed` when possible).
- Use nullable types (`?Type`) explicitly when `null` is a valid value.

**Constraints and Limitations**

- **Callback parameters**: Native PHP cannot express the signature of a `callable`; use `\Closure` or PHPDoc `@param callable(Foo): Bar` for documentation.
- **Generic collections**: Native PHP cannot express `array<string, User>`; use PHPDoc `@param array<string, User>` alongside the native `array` type.
- **Framework magic**: Some framework base classes use untyped magic methods; add types where you control the code, but leave framework internals alone.

#### Annotated Code Examples

**Example 1: Before and After Best Practices**

```php
<?php
// BEFORE: Poor practices — no types, global state, side effects
function calc($a, $b) {
    global $taxRate;
    echo $a + $b + $taxRate;
}

// AFTER: Best practices — types, no global, pure function
declare(strict_types=1);

function calculateTotal(float $subtotal, float $taxRate): float {
    return $subtotal + ($subtotal * $taxRate);
}

$total = calculateTotal(100.00, 0.20);
echo "Total: $total\n"; // Outputs: Total: 120
?>
```

**Expected Output:**
```
Total: 120
```

**Why This Output:**  
The "after" version uses type declarations (`float`), avoids global state, has no side effects, and returns a predictable value. It is testable and self-documenting.

---

**Example 2: Using Named Arguments for Boolean Flags**

```php
<?php
declare(strict_types=1);

function createUser(string $name, string $email, bool $sendWelcomeEmail = true): string {
    $emailStatus = $sendWelcomeEmail ? "Welcome email sent" : "No welcome email";
    return "User $name created. $emailStatus.\n";
}

// Positional call — what does "false" mean?
echo createUser("Alice", "alice@example.com", false);

// Named argument — self-documenting
echo createUser("Bob", "bob@example.com", sendWelcomeEmail: false);
?>
```

**Expected Output:**
```
User Alice created. No welcome email.
User Bob created. No welcome email.
```

**Why This Output:**  
Using a named argument `sendWelcomeEmail: false` makes the intent clear at the call site. A positional `false` is ambiguous; named arguments improve readability, especially for boolean flags.

---

**Example 3: Strict Types and Type Errors**

```php
<?php
declare(strict_types=1);

function addNumbers(int $a, int $b): int {
    return $a + $b;
}

try {
    echo addNumbers("5", 10);
} catch (TypeError $e) {
    echo "Type error caught: " . $e->getMessage() . "\n";
}
?>
```

**Expected Output:**
```
Type error caught: addNumbers(): Argument #1 ($a) must be of type int, string given
```

**Why This Output:**  
With `strict_types=1`, PHP enforces exact type matching. Passing a string `"5"` where an `int` is expected throws a `TypeError`, catching the mistake at the call site.

#### Real-World Cases

- **Laravel/Symfony Codebases**: Modern PHP frameworks enforce `declare(strict_types=1)` and full type declarations in their coding standards.
- **PHPStan/Psalm Adoption**: Projects using static analysis benefit from complete type declarations; untyped parameters are flagged as errors at level 5+.
- **API Client Libraries**: Libraries like Guzzle use named arguments in documentation and examples to clarify optional parameters, and type declarations to ensure valid inputs.

---

## References

**PHP Manual – Function parameters and arguments** - https://static.php.net/manual/en/functions.arguments.php

**PHP Manual – Type declarations** - https://static.php.net/manual/en/language.types.declarations.php

**PHP Manual – Type system (Intersection types)** - https://static.php.net/manual/en/language.types.system.php

**PHP Manual – New features (PHP 8.0)** - https://static.php.net/manual/en/migration80.new-features.php

**PHP RFC: Named Arguments** - https://wiki.php.net/rfc/named_params

**PSR-12: Extended Coding Style** - https://www.php-fig.org/psr/psr-12/

**PHP: Parameter Type Declarations (Best Practices)** - https://raw.githubusercontent.com/voku/agent-skills/84ecf946b83937fe7fcb1931432b13228467b4f2/skills/php-best-practices/rules/type-parameter-types.md

**PHP: Named arguments and variadic (Exakat)** - https://www.exakat.io/php-8-0-named-parameters-and-variadic/

**PHP: Intersection Types (Manual)** - https://raw.githubusercontent.com/armourinfosec/Secure-PHP-Development/main/Modern-PHP/Intersection-Types.md

**PHP: Default parameter values (Manual)** - https://static.php.net/manual/en/functions.arguments.php#functions.arguments.default