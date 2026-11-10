# C# Arithmetic Operators — Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
Arithmetic operators in C# are symbols that perform mathematical calculations on numeric operands. C# provides five binary arithmetic operators (`+`, `-`, `*`, `/`, `%`) for addition, subtraction, multiplication, division, and remainder, plus two unary operators (`++`, `--`) for incrementing and decrementing values by one. These operators form the foundation of virtually every computation in C# and follow the familiar rules of standard arithmetic, with important language-specific behaviours around integer division, remainder signs, and overflow.

**Technical Definition**
Arithmetic operators operate on operands of numeric types. The binary operators (`+`, `-`, `*`, `/`, `%`) take two operands, while the unary operators (`++`, `--`, unary `+`, unary `-`) take one. For the multiplicative operators (`*`, `/`, `%`), operands are converted to a common type via numeric promotion; the result is truncated or wrapped according to the type’s rules. Integer division truncates toward zero, and the remainder operator returns a result whose sign matches the dividend. In an unchecked context (the default), integral overflow silently wraps; in a checked context, it throws `OverflowException`. The `++` and `--` operators have distinct prefix and postfix forms that differ in the value they return. All arithmetic operators have well-defined precedence and associativity, and parentheses can override the default evaluation order.

**Beginner-Friendly Explanation**
Arithmetic operators are the **basic math symbols** you already know from school: `+` for adding, `-` for subtracting, `*` for multiplying, `/` for dividing, and `%` for the remainder (what is left over after division). The `++` and `--` operators are shortcuts that add or subtract `1`. C# mostly behaves like a calculator, but there are a few surprises: dividing two whole numbers gives a whole number (the fraction is thrown away), and `%` can give negative results. Understanding these quirks is essential to writing correct code.

---

### Key Characteristics

- **Five binary operators** — Addition (`+`), subtraction (`-`), multiplication (`*`), division (`/`), and remainder (`%`).
- **Two unary arithmetic operators** — Increment (`++`) and decrement (`--`), each with prefix and postfix forms.
- **Numeric promotion** — Operands of different numeric types are converted to a common type before the operation.
- **Integer division truncates toward zero** — `7 / 2` is `3`; `-7 / 2` is `-3` (not `-4`).
- **Remainder sign follows the dividend** — `-7 % 3` is `-1`; `7 % -3` is `1`.
- **Overflow depends on context** — Default `unchecked` context wraps silently; `checked` context throws `OverflowException`.
- **Well-defined precedence and associativity** — Multiplicative operators have higher precedence than additive; binary arithmetic operators are left-associative.

---

### Prerequisites

- Basic understanding of C# program structure (`class`, `Main`, `Console.WriteLine`).
- Familiarity with numeric data types (`int`, `double`, `decimal`, etc.) and their ranges.
- Knowledge of variables and assignment.
- Awareness of what an expression is and how values are computed.
- Basic understanding of integer vs. floating-point representation.

---

### Related Programming Areas

- **Data types and type conversion** — Arithmetic results depend heavily on the operand types and numeric promotion.
- **Control flow** — Conditions and loops rely on arithmetic expressions (counters, accumulators, bounds).
- **Financial and scientific computing** — `decimal` for money, `double` for science, and careful handling of precision.
- **Performance engineering** — Integer arithmetic vs. floating-point, and overflow checking, affect both speed and correctness.
- **Bit manipulation** — Remainder and division are related to bitwise operations for powers of two.

---

### Core Concepts / Features

1. Addition (`+`)
2. Subtraction (`-`)
3. Multiplication (`*`)
4. Division (`/`)
5. Remainder (`%`)
6. Increment (`++`)
7. Decrement (`--`)
8. Prefix vs. Postfix
9. Unary Operators
10. Operator Precedence
11. Operator Associativity
12. Best Practices

Each concept follows a uniform structure: definitions, purposes, syntax rules and structure, annotated examples with expected outputs, real-world cases, and references.

---

## 1. Addition (`+`)

### Definitions

**Core Definition**
The addition operator (`+`) computes the sum of two numeric operands.

**Technical Definition**
The binary `+` operator performs numeric addition. If both operands are integral types, the result is an integral sum; if either is a floating-point type (`float`, `double`) or `decimal`, the result follows that type’s rules. For `string` operands, `+` performs concatenation rather than addition. In a `checked` context, integral overflow throws `OverflowException`; in the default `unchecked` context, it wraps silently. The `+` operator is also used as a unary operator to represent identity (`+x`), which is rarely written explicitly.

**Beginner-Friendly Explanation**
`+` adds two numbers together. `10 + 3` gives `13`. It works for whole numbers, decimals, and even text (where it joins strings).

---

### Purposes

- To compute the sum of two numeric values.
- To accumulate totals in loops and aggregations.
- To implement counters and running sums.
- To concatenate strings (when operands are strings).
- To express unary identity (rarely used explicitly).

---

### Syntax Rules and Structure

```
result   =   leftOperand   +   rightOperand   ;
```

**Component breakdown**

- `leftOperand` — The first numeric (or string) value.
- `+` — The addition operator.
- `rightOperand` — The second numeric (or string) value.

#### Syntax Rules

- Both operands must be of numeric types (or strings for concatenation).
- Numeric promotion applies: `int + double` produces a `double`.
- Overflow behaviour depends on `checked`/`unchecked` context.

#### Constraints and Limitations

- Cannot add incompatible types (e.g., `int` and `DateTime`) without a conversion.
- String concatenation with `+` in loops can be inefficient; use `StringBuilder`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Integer Addition

```csharp
using System;

class Program
{
    static void Main()
    {
        int a = 10;
        int b = 3;
        int sum = a + b;
        Console.WriteLine(sum);   // 13
    }
}
```

**Expected Output**

```
13
```

**Why this output occurs**

- `10 + 3` evaluates to `13`, stored in `sum`.

---

#### Example 2: Mixed Numeric Types

```csharp
using System;

class Program
{
    static void Main()
    {
        int count = 5;
        double price = 2.5;
        double total = count + price;   // int promoted to double
        Console.WriteLine(total);       // 7.5
    }
}
```

**Expected Output**

```
7.5
```

**Why this output occurs**

- The `int` is promoted to `double`, and the addition is performed in floating point.

---

### Real-World Cases

- **Financial totals** — Summing line items in an invoice.
- **Game scores** — Accumulating points.
- **String building** — Concatenating messages (though interpolation is often preferred).

---

### References

- Arithmetic operators — C# reference — Addition operator - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators#addition-operator-
- C# operators — Addition and string concatenation - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/expressions/operators

---

## 2. Subtraction (`-`)

### Definitions

**Core Definition**
The subtraction operator (`-`) computes the difference between two numeric operands.

**Technical Definition**
The binary `-` operator subtracts the right operand from the left. Like addition, it follows numeric promotion rules and overflow behaviour based on context. As a unary operator, `-x` negates the value of `x`. For unsigned types, unary negation is not directly applicable without conversion.

**Beginner-Friendly Explanation**
`-` takes one number away from another. `10 - 3` gives `7`. It can also flip the sign of a number (`-5` becomes `5`).

---

### Purposes

- To compute the difference between two values.
- To decrement totals or balances.
- To calculate distances or deltas.
- To negate numeric values (unary minus).
- To implement countdowns and reverse loops.

---

### Syntax Rules and Structure

```
result   =   leftOperand   -   rightOperand   ;
```

**Component breakdown**

- `leftOperand` — The minuend.
- `-` — The subtraction operator.
- `rightOperand` — The subtrahend.

#### Syntax Rules

- Both operands must be numeric.
- Numeric promotion applies.
- Unary `-x` negates the operand.

#### Constraints and Limitations

- Subtracting unsigned types can wrap in unchecked contexts.
- Negating `int.MinValue` in a checked context throws `OverflowException`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Integer Subtraction

```csharp
using System;

class Program
{
    static void Main()
    {
        int total = 100;
        int spent = 30;
        int remaining = total - spent;
        Console.WriteLine(remaining);   // 70
    }
}
```

**Expected Output**

```
70
```

**Why this output occurs**

- `100 - 30` evaluates to `70`.

---

#### Example 2: Unary Minus

```csharp
using System;

class Program
{
    static void Main()
    {
        int temperature = 20;
        int cold = -temperature;   // negates
        Console.WriteLine(cold);   // -20
    }
}
```

**Expected Output**

```
-20
```

**Why this output occurs**

- Unary `-` flips the sign of `20` to `-20`.

---

### Real-World Cases

- **Banking** — Computing balances after withdrawals.
- **Physics** — Calculating displacement or velocity changes.
- **Countdowns** — Reverse loops and timers.

---

### References

- Arithmetic operators — C# reference — Subtraction operator - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators#subtraction-operator--
- Unary plus and minus operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators#unary-plus-and-minus-operators

---

## 3. Multiplication (`*`)

### Definitions

**Core Definition**
The multiplication operator (`*`) computes the product of two numeric operands.

**Technical Definition**
The binary `*` operator multiplies its operands. It is a multiplicative operator with higher precedence than additive operators. Numeric promotion applies, and overflow behaviour depends on `checked`/`unchecked` context. For `decimal`, overflow always throws regardless of context; for `float`/`double`, overflow produces infinity.

**Beginner-Friendly Explanation**
`*` multiplies two numbers. `10 * 3` gives `30`. It is like repeated addition.

---

### Purposes

- To compute products of values.
- To scale quantities (price × quantity).
- To calculate areas and volumes.
- To implement exponentiation by repeated multiplication.
- To convert units (e.g., inches to centimetres).

---

### Syntax Rules and Structure

```
result   =   leftOperand   *   rightOperand   ;
```

**Component breakdown**

- `leftOperand` — The multiplicand.
- `*` — The multiplication operator.
- `rightOperand` — The multiplier.

#### Syntax Rules

- Both operands must be numeric.
- Higher precedence than `+` and `-`.
- `decimal` overflow throws `OverflowException` even in `unchecked` context.

#### Constraints and Limitations

- Floating-point multiplication can lose precision.
- Integral overflow wraps in `unchecked` context.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Integer Multiplication

```csharp
using System;

class Program
{
    static void Main()
    {
        int width = 4;
        int height = 5;
        int area = width * height;
        Console.WriteLine(area);   // 20
    }
}
```

**Expected Output**

```
20
```

**Why this output occurs**

- `4 * 5` evaluates to `20`.

---

#### Example 2: Decimal Precision

```csharp
using System;

class Program
{
    static void Main()
    {
        decimal price = 19.99m;
        int quantity = 3;
        decimal total = price * quantity;
        Console.WriteLine(total);   // 59.97
    }
}
```

**Expected Output**

```
59.97
```

**Why this output occurs**

- `decimal` multiplication preserves exact base-10 precision.

---

### Real-World Cases

- **Retail** — Price × quantity.
- **Geometry** — Area and volume calculations.
- **Unit conversion** — Scaling factors.

---

### References

- Arithmetic operators — C# reference — Multiplication operator - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators#multiplication-operator-

---

## 4. Division (`/`)

### Definitions

**Core Definition**
The division operator (`/`) divides the left operand by the right operand. Its behaviour depends on whether the operands are integral or floating-point.

**Technical Definition**
When both operands are integers, `/` performs **integer division**: the result is truncated toward zero, discarding any fractional part. When at least one operand is a floating-point type (`float`, `double`, or `decimal`), `/` performs floating-point division. Division by zero throws `DivideByZeroException` for integral and `decimal` operands; for `float`/`double`, it produces positive or negative infinity (or `NaN` for `0.0 / 0.0`).

**Beginner-Friendly Explanation**
`/` divides one number by another. If both are whole numbers, the answer is also a whole number—the fraction is thrown away. `7 / 2` gives `3`, not `3.5`. To get a decimal answer, make at least one operand a decimal (e.g., `7.0 / 2`).

---

### Purposes

- To compute quotients.
- To distribute quantities evenly.
- To convert units (e.g., minutes to hours).
- To implement averaging (with careful attention to integer division).
- To split sequences into chunks.

---

### Syntax Rules and Structure

```
result   =   dividend   /   divisor   ;
```

**Component breakdown**

- `dividend` — The numerator.
- `/` — The division operator.
- `divisor` — The denominator.

#### Syntax Rules

- Integer division truncates toward zero: `7 / 2` = `3`; `-7 / 2` = `-3`.
- Floating-point division produces a fractional result.
- Integral division by zero throws `DivideByZeroException`.

#### Constraints and Limitations

- Integer division can surprise beginners who expect a fractional result.
- Division by zero must be guarded (except for floating-point types).

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Integer vs. Floating-Point Division

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine(7 / 2);       // 3 (integer division)
        Console.WriteLine(-7 / 2);      // -3 (truncates toward zero)
        Console.WriteLine(7.0 / 2);     // 3.5 (floating-point division)
    }
}
```

**Expected Output**

```
3
-3
3.5
```

**Why this output occurs**

- Integer division discards the fractional part and truncates toward zero.
- Floating-point division preserves the fraction.

---

#### Example 2: Division by Zero

```csharp
using System;

class Program
{
    static void Main()
    {
        try
        {
            int result = 10 / 0;
        }
        catch (DivideByZeroException)
        {
            Console.WriteLine("Cannot divide by zero");
        }
    }
}
```

**Expected Output**

```
Cannot divide by zero
```

**Why this output occurs**

- Integral division by zero throws `DivideByZeroException`.

---

### Real-World Cases

- **Averaging** — `sum / count` (with conversion to `double` for fractional averages).
- **Pagination** — Computing total pages.
- **Time conversion** — Seconds to minutes and hours.

---

### References

- Arithmetic operators — C# reference — Division operator - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators#division-operator-
- Integer division and remainder — C# fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/expressions/operators

---

## 5. Remainder (`%`)

### Definitions

**Core Definition**
The remainder operator (`%`) computes the remainder after dividing the left operand by the right operand.

**Technical Definition**
The `%` operator returns the remainder of `dividend / divisor`. For integral operands, the sign of the result matches the sign of the dividend: `-7 % 3` is `-1`, and `7 % -3` is `1`. For floating-point operands, `%` returns the floating-point remainder, which can also be negative. The remainder operator is useful for cycling through ranges and testing divisibility.

**Beginner-Friendly Explanation**
`%` gives you **what is left over** after division. `10 % 3` is `1` because `10` divided by `3` is `3` remainder `1`. It is perfect for checking if a number is even (`n % 2 == 0`) or for wrapping around a range.

---

### Purposes

- To compute the remainder of division.
- To test divisibility (even/odd, multiples).
- To cycle through a fixed range (e.g., `index % length`).
- To extract digits from a number.
- To implement wraparound behaviour.

---

### Syntax Rules and Structure

```
remainder   =   dividend   %   divisor   ;
```

**Component breakdown**

- `dividend` — The number being divided.
- `%` — The remainder operator.
- `divisor` — The number to divide by.

#### Syntax Rules

- Both operands must be numeric.
- For integers, the result has the same sign as the dividend.
- Division by zero throws `DivideByZeroException` for integral types.

#### Constraints and Limitations

- Negative operands can produce negative remainders; use `((n % m) + m) % m` to normalize.
- Floating-point `%` can produce unexpected results due to precision.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Basic Remainder

```csharp
using System;

class Program
{
    static void Main()
    {
        Console.WriteLine(10 % 3);    // 1
        Console.WriteLine(-7 % 3);    // -1
        Console.WriteLine(7 % -3);    // 1
    }
}
```

**Expected Output**

```
1
-1
1
```

**Why this output occurs**

- The remainder’s sign follows the dividend (the left operand).

---

#### Example 2: Even/Odd Check

```csharp
using System;

class Program
{
    static void Main()
    {
        int n = 7;
        Console.WriteLine(n % 2 == 0 ? "Even" : "Odd"); // Odd
    }
}
```

**Expected Output**

```
Odd
```

**Why this output occurs**

- `7 % 2` is `1`, so the number is odd.

---

### Real-World Cases

- **Cyclic buffers** — Wrapping indices.
- **Hashing** — Distributing values across buckets.
- **Game logic** — Turn-based systems (player `i % playerCount`).

---

### References

- Arithmetic operators — C# reference — Remainder operator - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators#remainder-operator-
- Remainder with negative operands — C# fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/expressions/operators

---

## 6. Increment (`++`)

### Definitions

**Core Definition**
The increment operator (`++`) increases its operand by `1`. It has prefix (`++x`) and postfix (`x++`) forms that differ in the value they return.

**Technical Definition**
The `++` operator adds `1` to its operand. In **prefix form** (`++x`), the operand is incremented first, and the expression evaluates to the new value. In **postfix form** (`x++`), the expression evaluates to the original value, and then the operand is incremented. The operator is defined for numeric and enum types, and can be overloaded for user-defined types. When used as a standalone statement, prefix and postfix have the same effect.

**Beginner-Friendly Explanation**
`++` adds `1` to a variable. `x++` means “use the current value, then add `1`.” `++x` means “add `1` first, then use the new value.” When `++` is on its own line, it does not matter which form you use.

---

### Purposes

- To increment counters and loop variables.
- To advance through sequences.
- To implement `for` loop updates concisely.
- To support prefix/postfix semantics in expressions.
- To provide a shorthand for `x = x + 1`.

---

### Syntax Rules and Structure

```
++variable   ;   // prefix increment
variable++   ;   // postfix increment
```

**Component breakdown**

- `++` — The increment operator.
- `variable` — A variable (not a constant or read-only field).

#### Syntax Rules

- Prefix returns the value after incrementing.
- Postfix returns the value before incrementing.
- The operand must be a variable (assignable expression).

#### Constraints and Limitations

- Cannot be applied to constants or read-only fields.
- Undefined behaviour can occur when combining `++` with other uses of the same variable in one expression (e.g., `x = x++`); avoid such constructs.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Prefix vs. Postfix

```csharp
using System;

class Program
{
    static void Main()
    {
        int x = 5;

        int prefix = ++x;    // increment first, then assign
        Console.WriteLine(prefix); // 6
        Console.WriteLine(x);      // 6

        x = 5;
        int postfix = x++;   // assign, then increment
        Console.WriteLine(postfix); // 5
        Console.WriteLine(x);       // 6
    }
}
```

**Expected Output**

```
6
6
5
6
```

**Why this output occurs**

- `++x` increments `x` to `6` and returns `6`.
- `x++` returns the original `5`, then increments `x` to `6`.

---

#### Example 2: Standalone Increment

```csharp
using System;

class Program
{
    static void Main()
    {
        int counter = 0;
        counter++;
        ++counter;
        Console.WriteLine(counter);   // 2
    }
}
```

**Expected Output**

```
2
```

**Why this output occurs**

- When used as standalone statements, both forms increment the variable.

---

### Real-World Cases

- **Loop counters** — `for (int i = 0; i < n; i++)`.
- **Pagination** — Advancing page numbers.
- **Game turns** — Moving to the next player.

---

### References

- Increment operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators#increment-operator-
- Prefix vs. postfix — C# fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/expressions/operators

---

## 7. Decrement (`--`)

### Definitions

**Core Definition**
The decrement operator (`--`) decreases its operand by `1`, with prefix and postfix forms that differ in the value they return.

**Technical Definition**
The `--` operator subtracts `1` from its operand. In prefix form (`--x`), the operand is decremented first, and the expression evaluates to the new value. In postfix form (`x--`), the expression evaluates to the original value, and then the operand is decremented. The operator is defined for numeric and enum types and can be overloaded.

**Beginner-Friendly Explanation**
`--` subtracts `1` from a variable. `x--` means “use the current value, then subtract `1`.” `--x` means “subtract `1` first, then use the new value.”

---

### Purposes

- To decrement counters and loop variables.
- To implement countdowns.
- To move backward through sequences.
- To provide a shorthand for `x = x - 1`.
- To support prefix/postfix semantics.

---

### Syntax Rules and Structure

```
--variable   ;   // prefix decrement
variable--   ;   // postfix decrement
```

**Component breakdown**

- `--` — The decrement operator.
- `variable` — A variable (not a constant).

#### Syntax Rules

- Prefix returns the value after decrementing.
- Postfix returns the value before decrementing.
- The operand must be a variable.

#### Constraints and Limitations

- Cannot be applied to constants.
- Avoid using both `--` and the same variable in the same expression in confusing ways.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Prefix vs. Postfix

```csharp
using System;

class Program
{
    static void Main()
    {
        int x = 5;

        int prefix = --x;    // decrement first, then assign
        Console.WriteLine(prefix); // 4
        Console.WriteLine(x);      // 4

        x = 5;
        int postfix = x--;   // assign, then decrement
        Console.WriteLine(postfix); // 5
        Console.WriteLine(x);       // 4
    }
}
```

**Expected Output**

```
4
4
5
4
```

**Why this output occurs**

- `--x` decrements `x` to `4` and returns `4`.
- `x--` returns `5`, then decrements `x` to `4`.

---

#### Example 2: Countdown Loop

```csharp
using System;

class Program
{
    static void Main()
    {
        for (int i = 3; i > 0; i--)
        {
            Console.WriteLine(i);
        }
    }
}
```

**Expected Output**

```
3
2
1
```

**Why this output occurs**

- The loop decrements `i` after each iteration until it reaches `0`.

---

### Real-World Cases

- **Countdowns** — Rockets, timers, reverse iteration.
- **Stack-like processing** — Popping items until empty.
- **Reverse loops** — Traversing arrays backward.

---

### References

- Decrement operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators#decrement-operator---
- Prefix vs. postfix — C# fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/expressions/operators

---

## 8. Prefix vs. Postfix

### Definitions

**Core Definition**
Prefix and postfix are the two forms of the `++` and `--` operators. They differ in **when** the increment or decrement occurs relative to the value returned by the expression.

**Technical Definition**
In **prefix** form (`++x`, `--x`), the operation is performed first, and the expression evaluates to the new value. In **postfix** form (`x++`, `x--`), the expression evaluates to the original value, and then the operation is performed. The distinction matters only when the result of the expression is used (e.g., in an assignment or as a method argument). When used as a standalone statement, both forms produce the same side effect. The C# standard does not specify the exact moment the “set” portion of an increment occurs within a larger expression, so combining `++`/`--` with other uses of the same variable in one expression (e.g., `x = x++`) is not recommended and can produce undefined results.

**Beginner-Friendly Explanation**
Think of `++` as “add one.” **Prefix** means “add first, then tell me the new number.” **Postfix** means “tell me the old number first, then add one.” If you are just writing `x++;` on its own line, the distinction does not matter.

---

### Purposes

- To control whether the old or new value is used in an expression.
- To implement idioms like `while (i-- > 0)`.
- To distinguish between “use then update” and “update then use” semantics.
- To enable correct indexing in expressions.

---

### Syntax Rules and Structure

```
++x   // prefix: increment, then return new value
x++   // postfix: return old value, then increment
--x   // prefix: decrement, then return new value
x--   // postfix: return old value, then decrement
```

**Component breakdown**

- `++` / `--` — The operators.
- `x` — The variable.
- Prefix form has the operator **before** the variable; postfix has it **after**.

#### Syntax Rules

- Prefix returns the new value.
- Postfix returns the original value.
- Standalone use makes the forms equivalent.

#### Constraints and Limitations

- Avoid using `++`/`--` on a variable that also appears elsewhere in the same expression; the result may be undefined or surprising.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Side-by-Side Comparison

```csharp
using System;

class Program
{
    static void Main()
    {
        int a = 5;
        int b = 5;

        Console.WriteLine(++a);   // 6 (increment first)
        Console.WriteLine(b++);   // 5 (use old, then increment)
        Console.WriteLine(b);     // 6
    }
}
```

**Expected Output**

```
6
5
6
```

**Why this output occurs**

- Prefix `++a` increments to `6` and returns `6`.
- Postfix `b++` returns `5`, then `b` becomes `6`.

---

#### Example 2: Postfix in a Condition

```csharp
using System;

class Program
{
    static void Main()
    {
        int i = 3;
        while (i-- > 0)
        {
            Console.WriteLine(i);
        }
    }
}
```

**Expected Output**

```
2
1
0
```

**Why this output occurs**

- The condition `i-- > 0` uses the current value of `i` and then decrements. The loop body runs for `i = 2, 1, 0`.

---

### Real-World Cases

- **Loop conditions** — `while (i-- > 0)` is a common idiom.
- **Indexing** — Using `arr[i++]` to fetch and advance.
- **State machines** — Transitioning after evaluating the old state.

---

### References

- Prefix vs. postfix increment and decrement — C# fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/expressions/operators
- Increment and decrement operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators

---

## 9. Unary Operators

### Definitions

**Core Definition**
Unary operators act on a single operand. In the context of arithmetic, the unary operators are unary plus (`+x`), unary minus (`-x`), increment (`++x`/`x++`), and decrement (`--x`/`x--`).

**Technical Definition**
Unary operators take one operand. The unary plus operator (`+x`) returns the value of its operand unchanged; it is rarely written explicitly. The unary minus operator (`-x`) negates the value. The increment and decrement operators are unary operators that add or subtract `1`. Unary operators have higher precedence than binary arithmetic operators. The logical negation operator (`!x`) is also unary but applies to `bool`, not numeric operands.

**Beginner-Friendly Explanation**
Unary operators work on **one thing at a time**. `-5` flips the sign, `+5` is just `5`, `++x` adds one, and `--x` subtracts one. They are the “single-operand” math tools.

---

### Purposes

- To negate numeric values.
- To explicitly indicate positive values (rarely needed).
- To increment or decrement by one.
- To combine with binary operators in complex expressions.
- To apply logical negation to booleans.

---

### Syntax Rules and Structure

```
+x   // unary plus (identity)
-x   // unary minus (negation)
++x  // prefix increment
--x  // prefix decrement
!x   // logical NOT (bool)
```

**Component breakdown**

- `+`, `-`, `++`, `--`, `!` — Unary operators.
- `x` — The operand.

#### Syntax Rules

- Unary operators have higher precedence than binary multiplicative and additive operators.
- Unary `+` leaves the value unchanged.
- Unary `-` negates the value.

#### Constraints and Limitations

- Unary `-` on `int.MinValue` in a checked context throws `OverflowException`.
- Logical negation (`!`) applies only to `bool`.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Unary Plus and Minus

```csharp
using System;

class Program
{
    static void Main()
    {
        int temperature = 20;
        int heatIndex = +temperature;   // unchanged
        int coldFactor = -temperature;  // negated

        Console.WriteLine(heatIndex);   // 20
        Console.WriteLine(coldFactor);  // -20
    }
}
```

**Expected Output**

```
20
-20
```

**Why this output occurs**

- Unary `+` is identity; unary `-` flips the sign.

---

#### Example 2: Combining Unary with Binary

```csharp
using System;

class Program
{
    static void Main()
    {
        int x = 5;
        int result = -x + 10;   // unary minus first, then addition
        Console.WriteLine(result); // 5
    }
}
```

**Expected Output**

```
5
```

**Why this output occurs**

- Unary `-x` is evaluated first (higher precedence), producing `-5`; then `-5 + 10` is `5`.

---

### Real-World Cases

- **Physics** — Negating vectors or directions.
- **Financial** — Representing debits as negative numbers.
- **Flags** — Toggling boolean states with `!`.

---

### References

- Unary plus and minus operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators#unary-plus-and-minus-operators
- Unary operators — C# fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/expressions/operators

---

## 10. Operator Precedence

### Definitions

**Core Definition**
Operator precedence determines the order in which operators in an expression are evaluated when they have different precedence levels.

**Technical Definition**
In an expression with multiple operators, operators with higher precedence are evaluated before operators with lower precedence. For arithmetic operators, the precedence order from highest to lowest is: unary operators (`+x`, `-x`, `++x`, `--x`) → multiplicative (`*`, `/`, `%`) → additive (`+`, `-`). This means `2 + 3 * 4` evaluates as `2 + (3 * 4) = 14`, not `(2 + 3) * 4 = 20`. Parentheses can be used to override precedence. The complete C# precedence table includes primary, unary, range, switch, multiplicative, additive, shift, relational, equality, logical AND, logical XOR, logical OR, conditional AND, conditional OR, null-coalescing, conditional, and assignment operators, from highest to lowest.

**Beginner-Friendly Explanation**
Precedence is the **math rule about what comes first**. Multiplication and division happen before addition and subtraction, just like in school. `2 + 3 * 4` is `14`, not `20`, because `*` has higher precedence than `+`. If you want a different order, use parentheses: `(2 + 3) * 4` is `20`.

---

### Purposes

- To define a consistent, predictable evaluation order.
- To allow concise expressions without excessive parentheses.
- To match mathematical conventions familiar to users.
- To enable the compiler to parse expressions unambiguously.
- To serve as a foundation for associativity rules.

---

### Syntax Rules and Structure (Precedence Order for Arithmetic)

```
Highest:  Unary + - ++ -- (and !, ~)
          Multiplicative * / %
          Additive + -
Lowest:   (Assignment)
```

**Component breakdown**

- **Unary** — Highest precedence among arithmetic operators.
- **Multiplicative** — `*`, `/`, `%` evaluate before `+` and `-`.
- **Additive** — `+`, `-` evaluate last among arithmetic operators.

#### Syntax Rules

- Higher precedence operators bind tighter.
- Parentheses override precedence.
- The full C# precedence table includes many other operator categories.

#### Constraints and Limitations

- Relying on precedence without parentheses can reduce readability; use parentheses for clarity when mixing operators.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Multiplication Before Addition

```csharp
using System;

class Program
{
    static void Main()
    {
        int result = 2 + 3 * 4;
        Console.WriteLine(result);   // 14
    }
}
```

**Expected Output**

```
14
```

**Why this output occurs**

- `3 * 4` (multiplicative, higher precedence) evaluates first to `12`; then `2 + 12` is `14`.

---

#### Example 2: Parentheses Override Precedence

```csharp
using System;

class Program
{
    static void Main()
    {
        int result = (2 + 3) * 4;
        Console.WriteLine(result);   // 20
    }
}
```

**Expected Output**

```
20
```

**Why this output occurs**

- Parentheses force `2 + 3` to evaluate first, producing `5`, which is then multiplied by `4`.

---

### Real-World Cases

- **Mathematical formulas** — Ensuring correct evaluation of expressions.
- **Financial calculations** — Interest computations where order matters.
- **Game physics** — Combining forces and accelerations.

---

### References

- Operator precedence — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/#operator-precedence
- Operator precedence table — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1242-operator-precedence-and-associativity

---

## 11. Operator Associativity

### Definitions

**Core Definition**
Associativity determines the order of evaluation when two operators have the **same precedence**. Left-associative operators evaluate from left to right; right-associative operators evaluate from right to left.

**Technical Definition**
When operators of equal precedence appear in an expression, associativity controls the grouping. The arithmetic binary operators (`*`, `/`, `%`, `+`, `-`) are **left-associative**: `a - b - c` is evaluated as `(a - b) - c`. The assignment operators and the conditional operator (`?:`) are **right-associative**: `a = b = c` is evaluated as `a = (b = c)`. Unary operators are right-associative by nature. Associativity can be overridden with parentheses.

**Beginner-Friendly Explanation**
Associativity is the **tie-breaker rule** when precedence cannot decide. If you have `10 - 3 - 2`, both `-` operators have the same precedence. Left-associativity means you go left to right: `(10 - 3) - 2 = 5`. If subtraction were right-associative, you would get `10 - (3 - 2) = 9`, which is not how math works.

---

### Purposes

- To resolve evaluation order for operators of equal precedence.
- To ensure consistent behaviour across expressions.
- To enable the familiar left-to-right reading of arithmetic.
- To define right-associativity for assignment chains.
- To support correct parsing of complex expressions.

---

### Syntax Rules and Structure

```
// Left-associative (arithmetic binary operators)
a   -   b   -   c      evaluates as   (a - b) - c

// Right-associative (assignment, conditional)
a   =   b   =   c      evaluates as   a = (b = c)
```

**Component breakdown**

- **Left-associative** — `*`, `/`, `%`, `+`, `-` (and most binary operators).
- **Right-associative** — Assignment operators (`=`, `+=`, etc.) and the conditional operator (`?:`).

#### Syntax Rules

- Arithmetic binary operators are left-associative.
- Assignment operators are right-associative.
- Parentheses override associativity.

#### Constraints and Limitations

- Mixing operators of different precedence without parentheses can obscure intent; use parentheses for clarity.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Left-Associative Subtraction

```csharp
using System;

class Program
{
    static void Main()
    {
        int result = 10 - 3 - 2;
        Console.WriteLine(result);   // 5
    }
}
```

**Expected Output**

```
5
```

**Why this output occurs**

- Left-associativity groups as `(10 - 3) - 2 = 7 - 2 = 5`.

---

#### Example 2: Right-Associative Assignment

```csharp
using System;

class Program
{
    static void Main()
    {
        int a, b, c;
        a = b = c = 5;   // right-associative: a = (b = (c = 5))
        Console.WriteLine($"{a}, {b}, {c}"); // 5, 5, 5
    }
}
```

**Expected Output**

```
5, 5, 5
```

**Why this output occurs**

- Assignment is right-associative, so `c = 5` happens first, then `b = 5`, then `a = 5`.

---

### Real-World Cases

- **Chained comparisons** — `a < b < c` is **not** valid C#; use `a < b && b < c` instead.
- **Fluent APIs** — Method chaining relies on left-associative member access.
- **Assignment chains** — `x = y = z` in initialization.

---

### References

- Operator associativity — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/#operator-associativity
- Associativity — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1242-operator-precedence-and-associativity

---

## 12. Best Practices

### Definitions

**Core Definition**
Arithmetic operator best practices are the guidelines that ensure calculations are correct, readable, and free from common pitfalls such as integer-division surprises, overflow, and precision loss.

**Technical Definition**
Best practices encompass using parentheses for clarity, understanding integer vs. floating-point division, handling remainder signs correctly, choosing `decimal` for financial calculations, guarding against division by zero, using `checked` contexts when overflow is unacceptable, and avoiding undefined behaviour such as combining `++`/`--` with other uses of the same variable in one expression.

**Beginner-Friendly Explanation**
Best practices are the habits that keep your math code from producing surprising results. Always check whether you are doing integer or decimal division, use parentheses to make your intention clear, and be careful with negative remainders.

---

### Purposes

- To prevent integer-division bugs.
- To handle negative remainders correctly.
- To avoid overflow and precision loss.
- To improve readability with parentheses.
- To write safe and predictable arithmetic code.

---

### Guidelines and Structure

#### General Guidelines

```
// Use parentheses for clarity
int result = (a + b) * c;   // not a + b * c

// Be explicit about integer vs. floating-point division
double average = (double)sum / count;   // not sum / count

// Normalize negative remainders if needed
int normalized = ((n % m) + m) % m;

// Use decimal for money
decimal total = price * quantity;

// Guard against division by zero
if (divisor != 0) { ... }

// Use checked for overflow-sensitive code
checked { int result = a * b; }
```

**Component breakdown**

- **Parentheses** — Clarify precedence and associativity.
- **Explicit division type** — Cast at least one operand to `double` or `decimal` for fractional results.
- **Remainder normalization** — Adjust for negative dividends.
- **`decimal`** — Exact base-10 arithmetic for currency.
- **Division-by-zero guards** — Check divisors before dividing.
- **`checked` context** — Detect overflow where it matters.

#### Rules and Constraints

- Always use parentheses when mixing operators with different precedence.
- Prefer `decimal` for financial calculations; `double` for scientific and general-purpose floating point.
- Avoid `x = x++` and similar constructs; behaviour can be undefined.
- Use `checked` for user-input arithmetic where overflow could be dangerous.
- Remember that integer division truncates toward zero, not toward negative infinity.

---

### Annotated Code Examples and Expected Outputs

#### Example 1: Avoiding Integer-Division Surprises

```csharp
using System;

class Program
{
    static void Main()
    {
        int sum = 7;
        int count = 2;

        int wrong = sum / count;                 // 3
        double right = (double)sum / count;       // 3.5

        Console.WriteLine(wrong);    // 3
        Console.WriteLine(right);    // 3.5
    }
}
```

**Expected Output**

```
3
3.5
```

**Why this output occurs**

- Integer division truncates; casting to `double` preserves the fraction.

---

#### Example 2: Normalizing Negative Remainders

```csharp
using System;

class Program
{
    static void Main()
    {
        int n = -7;
        int m = 3;

        int raw = n % m;                           // -1
        int normalized = ((n % m) + m) % m;        // 2

        Console.WriteLine(raw);          // -1
        Console.WriteLine(normalized);   // 2
    }
}
```

**Expected Output**

```
-1
2
```

**Why this output occurs**

- The raw remainder is negative; the normalized version produces a non-negative result in `[0, m)`.

---

### Real-World Cases

- **Financial systems** — Using `decimal` and checked arithmetic.
- **Scientific computing** — Using `double` and being aware of precision.
- **Game development** — Careful with remainder for cyclic indices.

---

### References

- Arithmetic operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators
- checked and unchecked statements — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/checked-and-unchecked
- IDE0047 and IDE0048: Parentheses preferences - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0047-ide0048
- SA1407: Arithmetic expressions should declare precedence - https://github.com/DotNetAnalyzers/StyleCopAnalyzers/blob/master/documentation/SA1407.md

---

## Deprecated, Unsafe, Implementation-Dependent, and Version-Specific Notes

| Feature | Status / Note |
|---------|---------------|
| Integer overflow (unchecked) | **Default behaviour**; wraps silently. Use `checked` to detect. |
| `decimal` overflow | **Always throws** `OverflowException`, even in `unchecked` context. |
| `float`/`double` overflow | Produces **infinity** (no exception). |
| `x = x++` | **Undefined behaviour**; the C# standard does not specify the result. Avoid. |
| Combining `++`/`--` with other uses of the same variable | **Not recommended**; evaluation order is unspecified. |
| `int.MinValue / -1` | In a `checked` context, throws `OverflowException`; in `unchecked`, wraps. |
| Division by zero (`float`/`double`) | Produces infinity or `NaN`; no exception. |
| Remainder with negative operands | Sign follows the dividend; **language-defined** (not implementation-dependent). |

---

## References (Consolidated)

- Arithmetic operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators
- C# operators — Operator precedence and associativity - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/#operator-precedence
- C# arithmetic, comparison, logical, and assignment operators — C# fundamentals - https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/expressions/operators
- Increment operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators#increment-operator-
- Decrement operator — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators#decrement-operator---
- Unary plus and minus operators — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/arithmetic-operators#unary-plus-and-minus-operators
- checked and unchecked statements — C# reference - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/checked-and-unchecked
- Operator precedence and associativity — C# language specification - https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1242-operator-precedence-and-associativity
- IDE0047 and IDE0048: Parentheses preferences - https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0047-ide0048
- SA1407: Arithmetic expressions should declare precedence - https://github.com/DotNetAnalyzers/StyleCopAnalyzers/blob/master/documentation/SA1407.md

---