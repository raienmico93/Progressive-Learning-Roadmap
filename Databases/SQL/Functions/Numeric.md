# SQL Numeric Functions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition**
SQL numeric functions are built-in routines provided by relational database management systems (RDBMS) that accept one or more numeric values (integers, decimals, floating-point numbers) as input and return a numeric result, enabling arithmetic computation, rounding, exponentiation, and random number generation directly within SQL statements.

**Technical Definition**
In SQL, numeric functions are scalar operators whose domain is numeric data types—`INTEGER`, `BIGINT`, `SMALLINT`, `NUMERIC`/`DECIMAL`, `REAL`, `DOUBLE PRECISION`, and `FLOAT`—and whose range is typically a numeric type whose precision and scale are determined by the input type and the function’s implementation. They are standardised in part by SQL:1999, SQL:2003, and SQL:2016, but each RDBMS (PostgreSQL, MySQL, SQL Server) implements a distinct set of functions with vendor-specific names, return types, and edge-case behaviours. Numeric functions operate under the rules of numeric type promotion, exact versus approximate arithmetic, and the database’s handling of overflow, rounding, and special values such as `NaN` and `Infinity`.

**Beginner-Friendly Explanation**
Think of numeric functions as a calculator built into your database. Just as a calculator lets you find absolute values, round numbers, calculate powers and square roots, or generate random numbers, SQL numeric functions let you perform these operations directly on data stored in database columns—without exporting the data to another program. You can compute totals, clean up decimal places, or add randomness to queries, all inside SQL.

---

### Key Characteristics

1. **Type-Sensitive Return Values**: Arithmetic functions such as `ABS`, `CEIL`, `FLOOR`, and `SIGN` generally return the same data type as their input, while transcendental functions such as `EXP`, `LN`, `LOG`, and `SQRT` cast their input to `FLOAT`/`DOUBLE PRECISION` and return floating-point results .
2. **Exact vs. Approximate Arithmetic**: `NUMERIC`/`DECIMAL` types perform exact arithmetic (recommended for monetary values), whereas `REAL` and `DOUBLE PRECISION` perform approximate binary floating-point arithmetic with inherent rounding error .
3. **Vendor-Specific Naming and Behaviour**: `CEIL` vs. `CEILING`, `TRUNC` vs. `TRUNCATE` (and SQL Server’s use of `ROUND` with a truncation flag), `RANDOM` vs. `RAND`, and the absence of `MOD()` in SQL Server are common portability pitfalls .
4. **Deterministic Except for Random Functions**: All mathematical functions except `RAND()`/`RANDOM()` are deterministic—they return the same result for the same input. `RAND()` is deterministic only when a seed is supplied .
5. **Overflow and Precision Limits**: Integer types have fixed ranges (`SMALLINT`, `INTEGER`, `BIGINT`), and exceeding them raises an overflow error. `NUMERIC`/`DECIMAL` types have user-defined precision and scale; exceeding the declared precision or scale triggers rounding or an overflow error depending on the RDBMS and configuration .
6. **NULL Propagation**: Most numeric functions return `NULL` when any required argument is `NULL`, consistent with SQL’s three-valued logic.

---

### Prerequisites

Before working with SQL numeric functions, you should understand:

- **Numeric data types**: The distinction between `INTEGER`, `BIGINT`, `NUMERIC`/`DECIMAL` (exact), and `REAL`/`DOUBLE PRECISION`/`FLOAT` (approximate).
- **Precision and scale**: How `NUMERIC(p, s)` defines the total number of digits (`p`) and the number of fractional digits (`s`).
- **Operator precedence**: How arithmetic operators (`+`, `-`, `*`, `/`, `%`) interact with function calls.
- **NULL semantics**: How `NULL` propagates through arithmetic expressions.
- **Basic mathematical concepts**: Absolute value, modulo, rounding, exponentiation, logarithms, and square roots.

---

### Related Programming Areas

| Area | Connection |
|------|------------|
| **Financial Systems** | `ROUND`, `TRUNC`, and exact `NUMERIC` arithmetic are essential for monetary calculations. |
| **Data Analysis & Reporting** | `CEIL`, `FLOOR`, `ABS`, and `MOD` support bucketing, deviation analysis, and parity checks. |
| **Scientific Computing** | `POWER`, `SQRT`, `EXP`, `LN`, and `LOG` implement mathematical models directly in SQL. |
| **Sampling & Simulation** | `RANDOM()`/`RAND()` supports random sampling, shuffling, and Monte Carlo simulations. |
| **Data Validation** | `SIGN` and range checks detect invalid or out-of-bound values. |
| **ETL & Data Cleaning** | `ROUND` and `TRUNC` standardise decimal precision across source systems. |

---

### Core Concepts / Features

The cheat sheet is organised into the following conceptual clusters:

1. **Basic Arithmetic & Sign** — `ABS`, `SIGN`, `MOD`
2. **Rounding & Truncation** — `CEIL`, `FLOOR`, `ROUND`, `TRUNC`
3. **Exponents, Roots & Logarithms** — `POWER`, `SQRT`, `EXP`, `LN`, `LOG`
4. **Constants & Random Generation** — `PI`, `RANDOM`
5. **Numeric Behavior & Limits** — Numeric precision, Numeric rounding, Numeric overflow

Each feature is documented with definitions, purposes beginning with “To”, syntax rules, constraints, annotated code examples with expected outputs, and real-world cases.

---

## 1. Basic Arithmetic & Sign

### 1.1 `ABS`

#### Definitions

**Core Definition**
`ABS` returns the absolute (non-negative) value of a number.

**Technical Definition**
`ABS(x)` is a scalar function that returns the magnitude of `x` without regard to sign. If `x` is negative, the result is `-x`; otherwise, the result is `x`. The return type is the same as the input type for integer and `NUMERIC` inputs; for floating-point inputs, it returns the same floating-point type .

**Beginner-Friendly Explanation**
`ABS` tells you “how far from zero” a number is, ignoring whether it is positive or negative. `ABS(-5)` and `ABS(5)` both give `5`.

#### Purposes

- To compute the magnitude of a deviation without regard to direction.
- To compare distances or differences where sign is irrelevant.
- To validate that a value lies within a tolerance band (e.g., `ABS(actual - expected) < 0.01`).
- To normalise negative values to positive ones for further processing.

#### Syntax Rules and Structure

**General Syntax**
```
ABS ( numeric_expression ) → numeric
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `ABS` | Function name. |
| `numeric_expression` | Any numeric type: integer, `NUMERIC`, `DECIMAL`, `REAL`, `DOUBLE PRECISION`, `FLOAT`. |
| `→ numeric` | Same type as input (for exact types) or floating-point type. |

**Syntax Rules**

1. `ABS(NULL)` returns `NULL`.
2. The result is always non-negative.
3. For `NUMERIC` inputs, the result is exact; for floating-point inputs, the result is approximate.

**Constraints and Limitations**

- `ABS` of the most negative integer value (e.g., `ABS(-2147483648)` for a 32-bit integer) can overflow in systems that cannot represent the positive counterpart, because the positive range is one less than the negative range. In PostgreSQL, `ABS` on the minimum integer value may raise an error or return an unexpected result depending on the type.

#### Annotated Code Examples

**Example 1: Absolute Value of a Negative Decimal**

```sql
SELECT ABS(-17.4) AS abs_decimal;
```

**Expected Output**

| abs_decimal |
|-------------|
| 17.4        |

**Why This Output Occurs**: `ABS` removes the negative sign, returning the magnitude `17.4` .

**Example 2: Absolute Value in a Tolerance Check**

```sql
SELECT order_id, ABS(actual_total - expected_total) AS deviation
FROM reconciliation
WHERE ABS(actual_total - expected_total) > 0.01;
```

**Expected Output (illustrative)**

| order_id | deviation |
|----------|-----------|
| 1001     | 0.05      |
| 1002     | 0.12      |

**Why This Output Occurs**: The `ABS` function ensures that deviations in either direction (over or under) are treated as positive magnitudes.

#### Real-World Cases

- **Financial reconciliation**: Measuring the absolute difference between recorded and expected amounts.
- **Quality control**: Checking whether a manufactured part’s dimension deviates from the target by more than a tolerance.
- **Game development**: Computing distance from a target without caring about direction.

---

### 1.2 `SIGN`

#### Definitions

**Core Definition**
`SIGN` returns an indicator of the sign of a number: `-1` for negative, `0` for zero, and `+1` for positive.

**Technical Definition**
`SIGN(x)` is a scalar function that returns `-1` if `x < 0`, `0` if `x = 0`, and `1` if `x > 0`. In PostgreSQL, the return type is `NUMERIC` for `NUMERIC` input and `DOUBLE PRECISION` for `DOUBLE PRECISION` input . In MySQL, `SIGN()` returns `-1`, `0`, or `1` depending on the sign of the argument .

**Beginner-Friendly Explanation**
`SIGN` tells you whether a number is positive, negative, or zero—it gives `1` for positive, `-1` for negative, and `0` for zero.

#### Purposes

- To categorise values as positive, negative, or zero without branching logic in application code.
- To implement conditional logic based on the direction of a change.
- To detect zero values explicitly.
- To simplify comparisons in `CASE` expressions.

#### Syntax Rules and Structure

**General Syntax**
```
SIGN ( numeric_expression ) → numeric
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `SIGN` | Function name. |
| `numeric_expression` | Any numeric expression. |
| `→ numeric` | `-1`, `0`, or `1` (type depends on input). |

**Syntax Rules**

1. `SIGN(NULL)` returns `NULL`.
2. `SIGN(0)` returns `0`.
3. `SIGN(-0.0)` returns `0` in most implementations.

**Constraints and Limitations**

- The return type may differ from the input type (e.g., PostgreSQL returns `NUMERIC` for `NUMERIC` input, not `INTEGER`).
- Behaviour with `NaN` (not-a-number) is implementation-specific; PostgreSQL may return `NaN` or raise an error.

#### Annotated Code Examples

**Example 1: Sign of a Negative Number**

```sql
SELECT SIGN(-8.4) AS sign_negative;
```

**Expected Output**

| sign_negative |
|---------------|
| -1            |

**Why This Output Occurs**: `-8.4` is less than zero, so `SIGN` returns `-1` .

**Example 2: Using `SIGN` to Classify Changes**

```sql
SELECT
  transaction_id,
  amount,
  CASE SIGN(amount)
    WHEN 1  THEN 'Credit'
    WHEN -1 THEN 'Debit'
    ELSE 'Zero'
  END AS transaction_type
FROM transactions;
```

**Expected Output (illustrative)**

| transaction_id | amount | transaction_type |
|----------------|--------|------------------|
| 1              | 100.00 | Credit           |
| 2              | -50.00 | Debit            |
| 3              | 0.00   | Zero             |

#### Real-World Cases

- **Accounting**: Classifying transactions as debits, credits, or zero-value entries.
- **Sensor data**: Determining whether a measurement is above, below, or at a baseline.
- **Game logic**: Determining the direction of movement or change.

---

### 1.3 `MOD`

#### Definitions

**Core Definition**
`MOD` returns the remainder of a division operation.

**Technical Definition**
`MOD(y, x)` (or `MOD(x, y)` depending on argument order in some systems) returns the remainder of `y / x`. In PostgreSQL, the function is `MOD(y, x)` and returns the remainder of `y/x`; it is available for `SMALLINT`, `INTEGER`, `BIGINT`, and `NUMERIC` . In MySQL, `MOD(N, M)` returns the remainder of `N` divided by `M`; the `%` operator is equivalent . SQL Server does **not** provide a `MOD()` function; use the `%` operator instead .

**Beginner-Friendly Explanation**
`MOD` tells you what is left over after division. `MOD(9, 4)` is `1` because 4 goes into 9 twice with 1 left over.

#### Purposes

- To determine whether a number is even or odd (`MOD(n, 2)`).
- To implement cyclic or repeating patterns (e.g., assigning items to buckets).
- To validate divisibility.
- To wrap values within a range (modular arithmetic).

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
MOD ( y numeric_type, x numeric_type ) → numeric_type
```
**MySQL Syntax**
```
MOD ( N, M )
```
or
```
N % M
```
**SQL Server Syntax**
```
dividend % divisor
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `y` / `N` / `dividend` | The number to be divided. |
| `x` / `M` / `divisor` | The divisor. |
| `→ numeric` | Remainder of the division. |

**Syntax Rules**

1. `MOD(y, 0)` raises a division-by-zero error.
2. `MOD(NULL, x)` returns `NULL`.
3. The sign of the result follows the sign of the dividend in most implementations (e.g., `MOD(-9, 4) = -1` in PostgreSQL).
4. In SQL Server, use `%` instead of `MOD()` .

**Constraints and Limitations**

- Not all systems support `MOD()` as a function; SQL Server requires `%`.
- Behaviour with negative operands varies by RDBMS (truncated vs. floored modulo).
- `MOD` with floating-point arguments is supported in MySQL but PostgreSQL’s `MOD` is intended for exact numeric types.

#### Annotated Code Examples

**Example 1: Remainder of Integer Division**

```sql
SELECT MOD(9, 4) AS remainder;
```

**Expected Output**

| remainder |
|-----------|
| 1         |

**Why This Output Occurs**: 9 divided by 4 is 2 with a remainder of 1 .

**Example 2: Even/Odd Classification with `MOD`**

```sql
SELECT
  employee_id,
  CASE MOD(employee_id, 2)
    WHEN 0 THEN 'Even'
    ELSE 'Odd'
  END AS parity
FROM employees;
```

**Expected Output (illustrative)**

| employee_id | parity |
|-------------|--------|
| 1           | Odd    |
| 2           | Even   |
| 3           | Odd    |

**Example 3: SQL Server `%` Operator**

```sql
SELECT 9 % 4 AS remainder;
```

**Expected Output**

| remainder |
|-----------|
| 1         |

**Why This Output Occurs**: SQL Server uses the `%` operator for modulo; there is no `MOD()` function .

#### Real-World Cases

- **Sharding**: `MOD(customer_id, 4)` assigns customers to one of four shards.
- **Scheduling**: Determining whether a date falls on a weekend or a specific cycle.
- **UI layout**: Alternating row colours (`MOD(row_number, 2)`).

---

## 2. Rounding & Truncation

### 2.1 `CEIL` / `CEILING`

#### Definitions

**Core Definition**
`CEIL` (or `CEILING`) returns the smallest integer greater than or equal to a given number.

**Technical Definition**
`CEIL(x)` returns the nearest integer that is not less than `x`. For positive non-integer values, it rounds up; for negative values, it rounds toward zero (e.g., `CEIL(-42.8) = -42`). PostgreSQL provides `CEIL` and `CEILING` as synonyms for `NUMERIC` and `DOUBLE PRECISION` inputs . MySQL provides `CEIL()` and `CEILING()` . SQL Server provides `CEILING()` .

**Beginner-Friendly Explanation**
`CEIL` rounds a number **up** to the next whole number, no matter how small the fractional part is. `CEIL(3.1)` is `4`; `CEIL(-3.1)` is `-3`.

#### Purposes

- To round quantities up to the nearest whole unit (e.g., boxes needed to pack items).
- To implement “ceiling” division in resource allocation.
- To align values to discrete intervals.
- To ensure that a value is never underestimated when rounding.

#### Syntax Rules and Structure

**General Syntax**
```
CEIL ( numeric_expression ) → numeric
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `CEIL` / `CEILING` | Function name (both accepted in most systems). |
| `numeric_expression` | Numeric value. |
| `→ numeric` | Smallest integer ≥ input. |

**Syntax Rules**

1. `CEIL(NULL)` returns `NULL`.
2. `CEIL(x)` returns the same type as the input (for exact types) or a floating-point type.
3. For negative values, `CEIL(-42.8)` returns `-42` because `-42` is the smallest integer greater than or equal to `-42.8` .

**Constraints and Limitations**

- Return type may differ between vendors (e.g., SQL Server’s `CEILING` returns the same data type as the input for exact types, but floating-point for approximate types).
- Not suitable for truncation (use `FLOOR` or `TRUNC`).

#### Annotated Code Examples

**Example 1: Ceiling of a Positive Decimal**

```sql
SELECT CEIL(42.8) AS ceiling_positive;
```

**Expected Output**

| ceiling_positive |
|------------------|
| 43               |

**Example 2: Ceiling of a Negative Decimal**

```sql
SELECT CEIL(-42.8) AS ceiling_negative;
```

**Expected Output**

| ceiling_negative |
|------------------|
| -42              |

**Why This Output Occurs**: `-42` is the smallest integer greater than or equal to `-42.8` .

**Example 3: Computing Boxes Needed**

```sql
SELECT CEIL(total_items / items_per_box::NUMERIC) AS boxes_needed
FROM shipments;
```

**Expected Output (illustrative)**

| boxes_needed |
|--------------|
| 5            |

**Why This Output Occurs**: If there are 41 items and 10 per box, `CEIL(4.1) = 5` boxes are needed.

#### Real-World Cases

- **Inventory management**: Determining the number of containers needed to hold a quantity.
- **Billing**: Rounding up billable hours to the nearest full hour.
- **Pagination**: Calculating the number of pages required to display all records.

---

### 2.2 `FLOOR`

#### Definitions

**Core Definition**
`FLOOR` returns the largest integer less than or equal to a given number.

**Technical Definition**
`FLOOR(x)` returns the nearest integer that is not greater than `x`. For positive values, it rounds down; for negative values, it rounds away from zero (e.g., `FLOOR(-42.8) = -43`). PostgreSQL provides `FLOOR` for `NUMERIC` and `DOUBLE PRECISION` . MySQL provides `FLOOR()` . SQL Server provides `FLOOR()` .

**Beginner-Friendly Explanation**
`FLOOR` rounds a number **down** to the previous whole number. `FLOOR(3.9)` is `3`; `FLOOR(-3.1)` is `-4`.

#### Purposes

- To round quantities down to the nearest whole unit (e.g., full weeks completed).
- To implement “floor” division.
- To bucket continuous values into discrete intervals.
- To ensure that a value is never overestimated when rounding.

#### Syntax Rules and Structure

**General Syntax**
```
FLOOR ( numeric_expression ) → numeric
```
**Syntax Rules**

1. `FLOOR(NULL)` returns `NULL`.
2. `FLOOR(x)` returns the same type as the input for exact types.
3. `FLOOR(-42.8) = -43` because `-43` is the largest integer less than or equal to `-42.8`.

**Constraints and Limitations**

- Same return-type caveats as `CEIL`.
- Not suitable for rounding to a specific number of decimal places (use `ROUND` for that).

#### Annotated Code Examples

**Example 1: Floor of a Positive Decimal**

```sql
SELECT FLOOR(42.8) AS floor_positive;
```

**Expected Output**

| floor_positive |
|----------------|
| 42             |

**Example 2: Floor of a Negative Decimal**

```sql
SELECT FLOOR(-42.8) AS floor_negative;
```

**Expected Output**

| floor_negative |
|----------------|
| -43            |

**Why This Output Occurs**: `-43` is the largest integer less than or equal to `-42.8` .

#### Real-World Cases

- **Age calculation**: `FLOOR(DATEDIFF(day, birth_date, CURRENT_DATE) / 365.25)` gives completed years.
- **Budgeting**: Determining how many full units can be purchased with a given budget.
- **Histogram binning**: Assigning values to the lower bound of each bin.

---

### 2.3 `ROUND`

#### Definitions

**Core Definition**
`ROUND` rounds a number to a specified number of decimal places (or to the nearest integer if no scale is given).

**Technical Definition**
`ROUND(numeric_expression [, scale])` returns `numeric_expression` rounded to `scale` decimal places. If `scale` is omitted, the value is rounded to the nearest integer. In PostgreSQL, for `NUMERIC` inputs, ties are broken by rounding away from zero; for `DOUBLE PRECISION`, tie-breaking is platform-dependent but typically “round half to even” . In SQL Server, `ROUND` accepts a third `function` argument: when `function` is `0` (default), the value is rounded; when `function` is nonzero, the value is truncated .

**Beginner-Friendly Explanation**
`ROUND` makes a number simpler by chopping off extra decimal places, but unlike `TRUNC`, it may change the last kept digit based on what comes after it.

#### Purposes

- To format monetary values to two decimal places.
- To reduce the precision of calculated results for reporting.
- To round measurements to a sensible number of significant figures.
- To implement “round half away from zero” or “round half to even” as required by business rules.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
ROUND ( numeric_expression ) → numeric
ROUND ( numeric_expression, scale ) → numeric
```
**MySQL Syntax**
```
ROUND ( X [, D] )
```
**SQL Server Syntax**
```
ROUND ( numeric_expression, length [, function] )
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `numeric_expression` | Value to round. |
| `scale` / `length` / `D` | Number of decimal places (positive) or places to the left of the decimal point (negative). |
| `function` (SQL Server) | `0` = round (default), nonzero = truncate. |

**Syntax Rules**

1. If `scale` is positive, rounding occurs to the right of the decimal point.
2. If `scale` is negative, rounding occurs to the left of the decimal point (e.g., `ROUND(1234.56, -1) = 1230`) .
3. `ROUND(NULL, ...)` returns `NULL`.
4. SQL Server’s third argument enables truncation: `ROUND(150.75, 0, 1)` returns `150.00` (truncated) .

**Constraints and Limitations**

- Tie-breaking behaviour differs between exact (`NUMERIC`) and approximate (`DOUBLE PRECISION`) types and between vendors.
- SQL Server’s `ROUND` is overloaded as both a rounding and truncation function.
- `ROUND` on a column in `WHERE` may prevent index usage.

#### Annotated Code Examples

**Example 1: Rounding to Two Decimal Places**

```sql
SELECT ROUND(42.4382, 2) AS rounded_two;
```

**Expected Output**

| rounded_two |
|-------------|
| 42.44       |

**Why This Output Occurs**: The third decimal digit is `8`, so `42.4382` rounds up to `42.44` .

**Example 2: Rounding to the Nearest Integer**

```sql
SELECT ROUND(42.4) AS rounded_integer;
```

**Expected Output**

| rounded_integer |
|-----------------|
| 42              |

**Example 3: SQL Server Truncation with `ROUND`**

```sql
SELECT ROUND(150.75, 0, 1) AS truncated;
```

**Expected Output (SQL Server)**

| truncated |
|-----------|
| 150.00    |

**Why This Output Occurs**: The third argument `1` tells SQL Server to truncate rather than round .

#### Real-World Cases

- **Invoice totals**: Rounding line-item amounts to two decimal places.
- **Statistical reporting**: Rounding percentages to one decimal place.
- **Grading systems**: Rounding final scores to the nearest integer.

---

### 2.4 `TRUNC` / `TRUNCATE`

#### Definitions

**Core Definition**
`TRUNC` (or `TRUNCATE`) truncates a number to a specified number of decimal places without rounding.

**Technical Definition**
`TRUNC(numeric_expression [, scale])` returns `numeric_expression` with all digits beyond `scale` decimal places removed, toward zero. PostgreSQL provides `TRUNC(numeric)` (truncate to integer) and `TRUNC(numeric, integer)` (truncate to `s` decimal places) . MySQL provides `TRUNCATE(X, D)`, which removes digits beyond `D` decimal places without rounding; if `D` is negative, digits to the left of the decimal point are zeroed . SQL Server does **not** have a `TRUNC`/`TRUNCATE` function; use `ROUND` with a nonzero third argument, or use `CAST`/`CONVERT` with integer arithmetic .

**Beginner-Friendly Explanation**
`TRUNC` chops off extra digits without caring about what comes after them. `TRUNC(42.8)` is `42`; `TRUNC(-42.8)` is `-42`. It never rounds up.

#### Purposes

- To remove fractional digits without changing the integer part (toward zero).
- To implement “floor toward zero” behaviour.
- To standardise decimal precision in ETL pipelines without introducing rounding bias.
- To emulate integer division in a controlled manner.

#### Syntax Rules and Structure

**PostgreSQL Syntax**
```
TRUNC ( numeric_expression ) → numeric
TRUNC ( numeric_expression, scale ) → numeric
```
**MySQL Syntax**
```
TRUNCATE ( X, D )
```
**SQL Server Workaround**
```
ROUND ( numeric_expression, length, 1 )  -- nonzero third arg truncates
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `numeric_expression` / `X` | Value to truncate. |
| `scale` / `D` / `length` | Number of decimal places to keep. |
| `→ numeric` | Truncated value. |

**Syntax Rules**

1. `TRUNC(x)` with no scale truncates to an integer toward zero .
2. `TRUNC(x, s)` keeps `s` decimal places and discards the rest .
3. In MySQL, negative `D` zeroes digits to the left of the decimal point .
4. SQL Server requires the `ROUND` workaround or explicit `CAST`.

**Constraints and Limitations**

- **Vendor-specific naming**: `TRUNC` (PostgreSQL), `TRUNCATE` (MySQL), and no native function in SQL Server .
- Truncation toward zero differs from `FLOOR` for negative numbers.
- Truncating a `NUMERIC` column with a declared scale may still cause rounding on insertion if the value exceeds the declared scale.

#### Annotated Code Examples

**Example 1: Truncating to an Integer**

```sql
SELECT TRUNC(42.8) AS trunc_integer;
```

**Expected Output**

| trunc_integer |
|---------------|
| 42            |

**Why This Output Occurs**: `TRUNC` removes the fractional part toward zero .

**Example 2: Truncating a Negative Number**

```sql
SELECT TRUNC(-42.8) AS trunc_negative;
```

**Expected Output**

| trunc_negative |
|----------------|
| -42            |

**Why This Output Occurs**: Truncation toward zero yields `-42`, unlike `FLOOR` which would return `-43` .

**Example 3: MySQL `TRUNCATE` with Decimal Places**

```sql
SELECT TRUNCATE(135.375, 2) AS truncated_two;
```

**Expected Output**

| truncated_two |
|---------------|
| 135.37        |

**Why This Output Occurs**: MySQL’s `TRUNCATE` removes digits beyond two decimal places without rounding .

**Example 4: SQL Server Truncation via `ROUND`**

```sql
SELECT ROUND(135.375, 2, 1) AS truncated_sql_server;
```

**Expected Output (SQL Server)**

| truncated_sql_server |
|----------------------|
| 135.37               |

**Why This Output Occurs**: The nonzero third argument forces truncation .

#### Real-World Cases

- **Payroll**: Truncating overtime hours to full minutes without rounding up.
- **Data warehousing**: Standardising currency amounts by truncating extra decimals.
- **Scientific measurement**: Discarding insignificant digits beyond the instrument’s precision.

---

## 3. Exponents, Roots & Logarithms

### 3.1 `POWER`

#### Definitions

**Core Definition**
`POWER` raises a number to a specified exponent.

**Technical Definition**
`POWER(base, exponent)` returns `base` raised to the power of `exponent`. PostgreSQL provides `POWER(a NUMERIC, b NUMERIC) → NUMERIC` and `POWER(a DOUBLE PRECISION, b DOUBLE PRECISION) → DOUBLE PRECISION` . MySQL provides `POWER(X, Y)` and `POW(X, Y)` . SQL Server provides `POWER(float_expression, y)` and returns a `FLOAT` .

**Beginner-Friendly Explanation**
`POWER` multiplies a number by itself a certain number of times. `POWER(2, 3)` is `2 × 2 × 2 = 8`.

#### Purposes

- To compute compound growth (e.g., compound interest).
- To calculate areas and volumes (`POWER(radius, 2)` for area).
- To implement exponential decay or growth models.
- To raise values to fractional powers (roots).

#### Syntax Rules and Structure

**General Syntax**
```
POWER ( base, exponent ) → numeric
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `base` | The number to be raised. |
| `exponent` | The power to raise the base to. |
| `→ numeric` | `base^exponent`. |

**Syntax Rules**

1. `POWER(NULL, ...)` returns `NULL`.
2. `POWER(0, 0)` returns `1` in PostgreSQL and MySQL; behaviour may vary in other systems.
3. Negative exponents return reciprocal powers.
4. Fractional exponents compute roots (e.g., `POWER(9, 0.5) = 3`).

**Constraints and Limitations**

- Return type depends on input types; SQL Server always returns `FLOAT`, which may introduce floating-point approximation .
- Large exponents can overflow numeric types.

#### Annotated Code Examples

**Example 1: Integer Power**

```sql
SELECT POWER(9, 3) AS power_result;
```

**Expected Output**

| power_result |
|--------------|
| 729          |

**Why This Output Occurs**: `9 × 9 × 9 = 729` .

**Example 2: Fractional Exponent (Square Root)**

```sql
SELECT POWER(9, 0.5) AS sqrt_via_power;
```

**Expected Output**

| sqrt_via_power |
|----------------|
| 3.0            |

**Why This Output Occurs**: `9` raised to `0.5` is the square root of `9`, which is `3`.

#### Real-World Cases

- **Finance**: Computing compound interest: `principal * POWER(1 + rate, years)`.
- **Geometry**: Calculating area or volume.
- **Machine learning**: Implementing polynomial features in SQL-based feature engineering.

---

### 3.2 `SQRT`

#### Definitions

**Core Definition**
`SQRT` returns the square root of a non-negative number.

**Technical Definition**
`SQRT(x)` returns the non-negative square root of `x`. PostgreSQL provides `SQRT(NUMERIC) → NUMERIC` and `SQRT(DOUBLE PRECISION) → DOUBLE PRECISION` . MySQL provides `SQRT(X)` . SQL Server provides `SQRT(float_expression)` and returns a `FLOAT` .

**Beginner-Friendly Explanation**
`SQRT` answers the question “what number, multiplied by itself, gives this value?” `SQRT(9)` is `3` because `3 × 3 = 9`.

#### Purposes

- To compute Euclidean distances.
- To calculate standard deviations (which involve square roots).
- To normalise vectors in scientific computations.
- To implement geometric formulas directly in SQL.

#### Syntax Rules and Structure

**General Syntax**
```
SQRT ( numeric_expression ) → numeric
```
**Syntax Rules**

1. `SQRT(NULL)` returns `NULL`.
2. `SQRT(-1)` raises an error or returns `NaN` depending on the RDBMS and type. In PostgreSQL, `SQRT(-1)` raises an error for `NUMERIC` input.
3. Return type is `NUMERIC` for `NUMERIC` input and `DOUBLE PRECISION`/`FLOAT` for approximate input.

**Constraints and Limitations**

- Negative inputs are invalid and cause errors or `NaN`.
- SQL Server always returns `FLOAT`, which may introduce approximation .

#### Annotated Code Examples

**Example 1: Square Root of 2**

```sql
SELECT SQRT(2) AS sqrt_two;
```

**Expected Output**

| sqrt_two          |
|-------------------|
| 1.4142135623730951 |

**Why This Output Occurs**: The square root of `2` is approximately `1.41421356…` .

**Example 2: Error on Negative Input**

```sql
-- PostgreSQL
SELECT SQRT(-1);
```

**Expected Output**: `ERROR: cannot take square root of a negative number`

#### Real-World Cases

- **Statistics**: Computing standard deviation: `SQRT(VAR_POP(x))`.
- **Geometry**: Calculating the hypotenuse: `SQRT(a*a + b*b)`.
- **Physics**: Computing magnitudes of vectors.

---

### 3.3 `EXP`

#### Definitions

**Core Definition**
`EXP` returns Euler’s number `e` raised to a given power.

**Technical Definition**
`EXP(x)` returns `e^x`, where `e ≈ 2.718281828…`. PostgreSQL provides `EXP(NUMERIC) → NUMERIC` and `EXP(DOUBLE PRECISION) → DOUBLE PRECISION` . MySQL provides `EXP(X)` . SQL Server provides `EXP(float_expression)` and returns a `FLOAT` .

**Beginner-Friendly Explanation**
`EXP` is the inverse of the natural logarithm. `EXP(1)` is approximately `2.718`, which is the value of `e`.

#### Purposes

- To compute continuous compound growth.
- To implement exponential decay models.
- To reverse natural logarithms in statistical computations.
- To model population growth or radioactive decay.

#### Syntax Rules and Structure

**General Syntax**
```
EXP ( numeric_expression ) → numeric
```
**Syntax Rules**

1. `EXP(NULL)` returns `NULL`.
2. `EXP(0) = 1`.
3. Return type is `NUMERIC` for `NUMERIC` input, otherwise `DOUBLE PRECISION`/`FLOAT`.

**Constraints and Limitations**

- Large arguments can overflow floating-point types.
- SQL Server returns `FLOAT`, which may introduce approximation .

#### Annotated Code Examples

**Example 1: `EXP(1)` Approximation of `e`**

```sql
SELECT EXP(1) AS e_approx;
```

**Expected Output**

| e_approx          |
|-------------------|
| 2.718281828459045 |

**Why This Output Occurs**: `e^1 = e ≈ 2.718281828…` .

#### Real-World Cases

- **Finance**: Continuous compounding: `principal * EXP(rate * time)`.
- **Biology**: Population growth models.
- **Physics**: Radioactive decay: `N0 * EXP(-lambda * t)`.

---

### 3.4 `LN`

#### Definitions

**Core Definition**
`LN` returns the natural logarithm (base `e`) of a positive number.

**Technical Definition**
`LN(x)` returns the logarithm of `x` to base `e`. PostgreSQL provides `LN(NUMERIC) → NUMERIC` and `LN(DOUBLE PRECISION) → DOUBLE PRECISION` . MySQL provides `LN(X)` .

**Beginner-Friendly Explanation**
`LN` answers “to what power must `e` be raised to get this number?” `LN(e)` is `1`.

#### Purposes

- To linearise exponential relationships for analysis.
- To compute continuous growth rates.
- To invert `EXP` in statistical modelling.
- To calculate information entropy.

#### Syntax Rules and Structure

**General Syntax**
```
LN ( numeric_expression ) → numeric
```
**Syntax Rules**

1. `LN(NULL)` returns `NULL`.
2. `LN(x)` for `x ≤ 0` raises an error in PostgreSQL (`cannot take logarithm of a non-positive number`).
3. `LN(1) = 0`.

**Constraints and Limitations**

- Domain is strictly positive numbers.
- SQL Server uses `LOG(float_expression)` for natural logarithm (with one argument) .

#### Annotated Code Examples

**Example 1: Natural Logarithm of 2**

```sql
SELECT LN(2) AS ln_two;
```

**Expected Output**

| ln_two             |
|--------------------|
| 0.6931471805599453 |

**Why This Output Occurs**: `e^0.693147… ≈ 2` .

#### Real-World Cases

- **Finance**: Computing continuously compounded returns.
- **Data science**: Log-transforming skewed data for regression.
- **Information theory**: Calculating entropy.

---

### 3.5 `LOG`

#### Definitions

**Core Definition**
`LOG` returns the logarithm of a number to a specified base (or base 10 if no base is specified, depending on the RDBMS).

**Technical Definition**
`LOG` has vendor-specific semantics. In PostgreSQL, `LOG(numeric)` returns the base-10 logarithm; `LOG(b numeric, x numeric)` returns the logarithm of `x` to base `b` . In MySQL, `LOG(X)` returns the natural logarithm (same as `LN(X)`), `LOG(B, X)` returns the logarithm of `X` to base `B`, and `LOG10(X)` returns the base-10 logarithm . SQL Server provides `LOG(float_expression [, base])`: with one argument it returns the natural logarithm; with two arguments it returns the logarithm to the specified base .

**Beginner-Friendly Explanation**
`LOG` tells you “to what power must a given base be raised to produce this number?” The base can be `10`, `2`, `e`, or any other positive number not equal to `1`.

#### Purposes

- To compute logarithms in any base for scientific and financial calculations.
- To measure orders of magnitude (base-10 logarithms).
- To implement information-theoretic measures (base-2 logarithms).
- To reverse exponentiation in `POWER` calculations.

#### Syntax Rules and Structure

**PostgreSQL**
```
LOG ( numeric_expression ) → numeric          -- base 10
LOG ( base numeric, numeric_expression ) → numeric
```
**MySQL**
```
LOG ( X )          -- natural logarithm
LOG ( B, X )       -- logarithm of X to base B
LOG10 ( X )        -- base 10
```
**SQL Server**
```
LOG ( float_expression )          -- natural logarithm
LOG ( float_expression, base )    -- logarithm to base
```

**Syntax Rules**

1. `LOG(NULL, ...)` returns `NULL`.
2. Base must be positive and not equal to `1`.
3. Argument must be positive; otherwise an error or `NULL` is returned depending on the system.

**Constraints and Limitations**

- **Significant vendor divergence**: PostgreSQL’s single-argument `LOG` is base 10, MySQL’s is natural log, and SQL Server’s is natural log.
- Return types are floating-point or `NUMERIC` depending on input.

#### Annotated Code Examples

**Example 1: Base-10 Logarithm (PostgreSQL)**

```sql
SELECT LOG(100) AS log_base10;
```

**Expected Output**

| log_base10 |
|------------|
| 2          |

**Why This Output Occurs**: `10^2 = 100` .

**Example 2: Logarithm to a Custom Base**

```sql
-- PostgreSQL
SELECT LOG(2.0, 64.0) AS log_base2;
```

**Expected Output**

| log_base2 |
|-----------|
| 6.0       |

**Why This Output Occurs**: `2^6 = 64` .

**Example 3: MySQL `LOG` with Two Arguments**

```sql
SELECT LOG(2, 64) AS log_base2_mysql;
```

**Expected Output**

| log_base2_mysql |
|-----------------|
| 6               |

#### Real-World Cases

- **Finance**: Computing the number of periods needed for an investment to reach a target.
- **Computer science**: Measuring algorithm complexity in base 2.
- **Earthquake science**: Richter scale is logarithmic (base 10).

---

## 4. Constants & Random Generation

### 4.1 `PI`

#### Definitions

**Core Definition**
`PI` returns the mathematical constant π (pi), approximately `3.141592653589793`.

**Technical Definition**
`PI()` is a niladic function that returns the value of π as a `DOUBLE PRECISION` (PostgreSQL) or `FLOAT` (SQL Server/MySQL). In PostgreSQL, `PI()` returns `DOUBLE PRECISION` . MySQL provides `PI()` .

**Beginner-Friendly Explanation**
`PI` simply gives you the value of π, the ratio of a circle’s circumference to its diameter, so you don’t have to type it out manually.

#### Purposes

- To compute areas and circumferences of circles.
- To perform trigonometric calculations.
- To convert between radians and degrees.
- To avoid hard-coding the value of π in SQL expressions.

#### Syntax Rules and Structure

**General Syntax**
```
PI ( ) → double precision
```
**Syntax Rules**

1. `PI()` takes no arguments.
2. Returns an approximate value (not an exact symbolic π).

**Constraints and Limitations**

- Returns an approximate floating-point value; precision depends on the underlying type.

#### Annotated Code Examples

**Example 1: Area of a Circle**

```sql
SELECT PI() * POWER(5, 2) AS area_of_circle;
```

**Expected Output**

| area_of_circle    |
|-------------------|
| 78.53981633974483 |

**Why This Output Occurs**: Area = π × r² = π × 25 ≈ 78.5398… .

#### Real-World Cases

- **Geometry calculations**: Area, circumference, and volume of circular or spherical objects.
- **Signal processing**: Trigonometric transforms that require π.
- **Physics**: Angular frequency calculations.

---

### 4.2 `RANDOM` / `RAND`

#### Definitions

**Core Definition**
`RANDOM` (PostgreSQL) or `RAND` (MySQL, SQL Server) returns a pseudo-random floating-point number between `0.0` (inclusive) and `1.0` (exclusive).

**Technical Definition**
`RANDOM()` in PostgreSQL returns a uniformly distributed `DOUBLE PRECISION` value in the range `0.0 ≤ x < 1.0` . MySQL’s `RAND()` returns a random floating-point value `v` in the range `0 ≤ v < 1.0`; when a seed is supplied, it produces a repeatable sequence . SQL Server’s `RAND([seed])` returns a pseudo-random `FLOAT` from `0` through `1`, exclusive; with a seed, repeated calls return the same sequence . All three use deterministic pseudo-random number generators (PRNGs) that are not suitable for cryptographic purposes .

**Beginner-Friendly Explanation**
`RANDOM` generates a random decimal between 0 and 1. You can multiply it by a range and round to get random integers—useful for sampling or testing.

#### Purposes

- To select random rows for sampling or testing.
- To generate random values for simulation or Monte Carlo methods.
- To shuffle query results randomly.
- To create unique identifiers or nonces (non-cryptographic).

#### Syntax Rules and Structure

**PostgreSQL**
```
RANDOM ( ) → double precision
```
(PostgreSQL 17+ also supports `random(min, max)` for ranges .)

**MySQL**
```
RAND ( [seed] ) → double
```
**SQL Server**
```
RAND ( [seed] ) → float
```

**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `seed` (optional) | Integer seed for repeatable sequences. |
| `→ numeric` | Random value in `[0, 1)`. |

**Syntax Rules**

1. Without a seed, each call typically produces a different value (except within a single SQL statement in some systems where `RAND()` may be evaluated once).
2. With a seed, the sequence becomes deterministic and repeatable.
3. `RANDOM()` takes no arguments in PostgreSQL (pre-17); MySQL and SQL Server accept an optional seed.
4. In SQL Server, `RAND()` in an `UPDATE`/`INSERT` gives the same value to all affected rows .

**Constraints and Limitations**

- **Not cryptographically secure**: PRNGs are deterministic and predictable .
- `RAND()` in `WHERE` clauses may be evaluated per row, preventing index optimisation .
- Seed behaviour differs across RDBMS.
- PostgreSQL 17 introduced range-based `random(min, max)` for integers and numerics .

#### Annotated Code Examples

**Example 1: Basic Random Value (PostgreSQL)**

```sql
SELECT RANDOM() AS random_value;
```

**Expected Output (illustrative)**

| random_value      |
|-------------------|
| 0.897124072839091 |

**Why This Output Occurs**: `RANDOM()` returns a value in `[0, 1)` .

**Example 2: Random Integer in a Range (MySQL)**

```sql
SELECT FLOOR(1 + RAND() * 10) AS random_1_to_10;
```

**Expected Output (illustrative)**

| random_1_to_10 |
|----------------|
| 7              |

**Why This Output Occurs**: `RAND()` gives `0 ≤ v < 1`; multiplying by `10` gives `0 ≤ v < 10`; adding `1` gives `1 ≤ v < 11`; `FLOOR` yields integers `1` through `10` .

**Example 3: Repeatable Sequence with Seed (SQL Server)**

```sql
SELECT RAND(123) AS seeded_random;
```

**Expected Output (SQL Server, illustrative)**

| seeded_random     |
|-------------------|
| 0.713591993212923 |

**Why This Output Occurs**: With a seed, `RAND` produces a deterministic sequence .

#### Real-World Cases

- **Random sampling**: `SELECT * FROM table ORDER BY RANDOM() LIMIT 100;`.
- **A/B testing**: Randomly assigning users to test groups.
- **Games**: Rolling dice or shuffling cards in a database-backed game.
- **Test data generation**: Populating test tables with random values.

---

## 5. Numeric Behavior & Limits

### 5.1 Numeric Precision

#### Definitions

**Core Definition**
Numeric precision refers to the total number of significant digits that a numeric value or column can represent.

**Technical Definition**
In SQL, precision is the total count of significant digits in a number, both to the left and to the right of the decimal point. For the `NUMERIC(p, s)` / `DECIMAL(p, s)` type, `p` is the precision and `s` is the scale (the number of fractional digits). For example, `23.5141` has a precision of 6 and a scale of 4 . PostgreSQL’s `NUMERIC` can store up to 131,072 digits before the decimal point and up to 16,383 digits after the decimal point (unconstrained) . SQL Server’s `DECIMAL` has a maximum precision of 38 digits .

**Beginner-Friendly Explanation**
Precision is the total number of digits a number can hold. A `NUMERIC(5, 2)` column can store numbers with 5 total digits, 2 of which are after the decimal point—so values like `123.45` fit, but `1234.56` would overflow.

#### Purposes

- To define exact storage requirements for monetary and scientific data.
- To prevent accidental loss of precision in calculations.
- To enforce data quality by rejecting values that exceed defined limits.
- To ensure portability across systems with different default precisions.

#### Syntax Rules and Structure

**Column Declaration**
```
NUMERIC ( precision, scale )
DECIMAL ( precision, scale )
```
**Component Breakdown**

| Component | Description |
|-----------|-------------|
| `precision` | Total number of significant digits. |
| `scale` | Number of fractional digits. |

**Syntax Rules**

1. `scale` must be between `0` and `precision` (SQL standard); PostgreSQL allows a wider range including negative scale .
2. `NUMERIC` without precision/scale is “unconstrained” and stores values of arbitrary precision up to implementation limits .
3. If a value exceeds the declared scale, it is rounded to the declared scale; if it exceeds the declared precision minus scale (digits to the left), an overflow error is raised .

**Constraints and Limitations**

- Maximum precision varies: PostgreSQL unconstrained `NUMERIC` allows up to 131,072 digits before the decimal point; SQL Server `DECIMAL` maxes out at 38 digits .
- Using unconstrained `NUMERIC` may have performance implications.
- Floating-point types (`REAL`, `DOUBLE PRECISION`) have binary precision (e.g., 6 and 15 decimal digits, respectively) that is approximate .

#### Annotated Code Examples

**Example 1: Declaring a `NUMERIC(5, 2)` Column**

```sql
CREATE TABLE monetary_values (
  amount NUMERIC(5, 2)
);

INSERT INTO monetary_values VALUES (123.45);  -- succeeds
-- INSERT INTO monetary_values VALUES (1234.56); -- overflow error
```

**Explanation**: `NUMERIC(5, 2)` allows 3 digits before the decimal and 2 after. `123.45` fits (3 + 2 = 5 digits); `1234.56` has 4 digits before the decimal, exceeding the allowed 3, and triggers an overflow error .

**Example 2: Rounding to Declared Scale**

```sql
INSERT INTO monetary_values VALUES (123.456);
-- Stored as 123.46 (rounded to 2 decimal places)
```

**Explanation**: When the scale of the input exceeds the declared scale, the value is rounded, not rejected (in PostgreSQL and most systems) .

#### Real-World Cases

- **Financial systems**: Storing currency amounts as `NUMERIC(12, 2)` to support values up to 9,999,999,999.99.
- **Scientific data**: Using unconstrained `NUMERIC` for high-precision measurements.
- **Data migration**: Aligning precision across source and target systems to avoid truncation.

---

### 5.2 Numeric Rounding

#### Definitions

**Core Definition**
Numeric rounding is the process of reducing the number of significant digits in a number to fit a target precision or scale, according to a defined rule.

**Technical Definition**
SQL rounding behaviour depends on the function (`ROUND`, `CEIL`, `FLOOR`, `TRUNC`) and the data type. For exact numeric types (`NUMERIC`/`DECIMAL`), rounding typically follows “round half away from zero” in PostgreSQL . For approximate types (`DOUBLE PRECISION`), tie-breaking is platform-dependent, commonly “round half to even” (banker’s rounding) . In SQL Server, `ROUND` follows “round half away from zero” by default, and the third argument can force truncation . MySQL’s `ROUND` follows “round half away from zero” for exact types and “round half to even” for floating-point values .

**Beginner-Friendly Explanation**
Rounding is how a database decides what to do when a number has more digits than you want to keep. Different functions and data types may use different rules, especially when the digit being removed is exactly `5`.

#### Purposes

- To format values for display or storage without losing meaningful precision.
- To comply with accounting standards that prescribe specific rounding rules.
- To avoid bias in statistical calculations by choosing an appropriate tie-breaking rule.
- To ensure consistent results across different data types.

#### Syntax Rules and Structure

Rounding is controlled by:

1. **Functions**: `ROUND`, `CEIL`, `FLOOR`, `TRUNC`.
2. **Data type**: Exact vs. approximate numeric types.
3. **Column scale**: `NUMERIC(p, s)` coerces values to `s` decimal places.
4. **Vendor-specific settings**: e.g., SQL Server’s `SET NUMERIC_ROUNDABORT` .

**Constraints and Limitations**

- Tie-breaking rules differ across RDBMS and data types.
- Rounding at the column level may silently alter data on insertion.
- Using `ROUND` on indexed columns in `WHERE` can prevent index usage.

#### Annotated Code Examples

**Example 1: PostgreSQL Exact vs. Approximate Tie-Breaking**

```sql
-- NUMERIC: round half away from zero
SELECT ROUND(2.5::NUMERIC) AS numeric_round;   -- returns 3

-- DOUBLE PRECISION: typically round half to even
SELECT ROUND(2.5::DOUBLE PRECISION) AS float_round;  -- may return 2 or 3
```

**Explanation**: For `NUMERIC`, PostgreSQL rounds `2.5` to `3` (away from zero) . For `DOUBLE PRECISION`, tie-breaking is platform-dependent and often “round to nearest even,” which may return `2` .

**Example 2: SQL Server `ROUND` with Truncation Flag**

```sql
SELECT
  ROUND(150.75, 0)    AS rounded,    -- 151.00
  ROUND(150.75, 0, 1) AS truncated;  -- 150.00
```

**Explanation**: The third argument `1` forces truncation, while the default `0` rounds .

#### Real-World Cases

- **Accounting**: Ensuring that rounding of monetary amounts complies with GAAP or IFRS rules.
- **Statistical reporting**: Using banker’s rounding to minimise cumulative bias.
- **Data warehousing**: Applying consistent rounding rules across ETL transformations.

---

### 5.3 Numeric Overflow

#### Definitions

**Core Definition**
Numeric overflow occurs when a computed value exceeds the maximum (or minimum) value representable by the target data type.

**Technical Definition**
In SQL, overflow occurs when an arithmetic operation or type cast produces a value outside the range of the destination type. Integer types have fixed ranges (`SMALLINT`: -32,768 to 32,767; `INTEGER`: -2,147,483,648 to 2,147,483,647; `BIGINT`: -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807) . For `NUMERIC`/`DECIMAL`, overflow occurs when the number of digits to the left of the decimal point exceeds `precision - scale`; for example, `NUMERIC(5, 2)` allows only 3 digits before the decimal, so `1000.00` overflows . Overflow handling varies: PostgreSQL and SQL Server raise an error; MySQL may raise an error in strict mode or clamp to the maximum value in non-strict mode .

**Beginner-Friendly Explanation**
Overflow happens when a number is too big for its container—like trying to pour 2 litres of water into a 1-litre bottle. The database either raises an error or, in some configurations, silently stores a wrong value.

#### Purposes

- To detect and prevent silent data corruption from out-of-range values.
- To design schemas with sufficient range for expected data.
- To implement graceful error handling in ETL pipelines.
- To choose appropriate data types that balance storage and range.

#### Syntax Rules and Structure

Overflow can arise from:

1. **Arithmetic operations**: `2147483647 + 1` on an `INTEGER` column.
2. **Aggregations**: `SUM()` of a large `INTEGER` column may overflow even if individual values fit.
3. **Type casts**: Casting a large `BIGINT` to `INTEGER`.
4. **Column insertion**: Inserting a value exceeding `NUMERIC(p, s)` limits.
5. **Function results**: `POWER(10, 10)` cast to `INTEGER`.

**Constraints and Limitations**

- Overflow behaviour is vendor-specific: PostgreSQL raises `ERROR: integer out of range`; MySQL raises `ERROR 1690` in strict mode; SQL Server raises “Arithmetic overflow error” .
- `SUM()` of `INTEGER` columns returns `INTEGER` in many systems, so the sum can overflow even if each row fits. Cast to `BIGINT` or `NUMERIC` before summing .
- `NUMERIC` with declared precision raises an error on overflow; unconstrained `NUMERIC` has much larger limits but still has implementation bounds .

#### Annotated Code Examples

**Example 1: Integer Overflow in PostgreSQL**

```sql
SELECT 2147483647 + 1;
```

**Expected Output**: `ERROR: integer out of range`

**Why This Output Occurs**: The maximum `INTEGER` value is 2,147,483,647; adding 1 exceeds the range .

**Example 2: `NUMERIC` Precision Overflow**

```sql
CREATE TABLE t (amount NUMERIC(5, 2));
INSERT INTO t VALUES (1000.00);
```

**Expected Output (PostgreSQL)**: `ERROR: numeric field overflow`

**Why This Output Occurs**: `NUMERIC(5, 2)` allows 3 digits before the decimal; `1000.00` has 4, so it overflows .

**Example 3: Avoiding `SUM()` Overflow**

```sql
-- Risky: SUM of INTEGER may overflow
SELECT SUM(sales_amount) FROM sales;

-- Safe: cast to BIGINT or NUMERIC before summing
SELECT SUM(sales_amount::BIGINT) FROM sales;
```

**Explanation**: `SUM(INTEGER)` returns `INTEGER` in PostgreSQL and may overflow if the total exceeds 2,147,483,647. Casting to `BIGINT` extends the range .

#### Real-World Cases

- **Financial aggregation**: Summing large transaction volumes where individual amounts fit in `INTEGER` but the total exceeds `INTEGER` range.
- **Data migration**: Moving data from a system with smaller integer types to one with larger types, or vice versa.
- **Scientific computing**: Ensuring that intermediate calculations do not overflow before the final result is cast.

---

## References

1. **PostgreSQL 16 Documentation — Mathematical Functions and Operators** — https://www.postgresql.org/docs/16/functions-math.html
2. **PostgreSQL 16 Documentation — Numeric Types** — https://www.postgresql.org/docs/16/datatype-numeric.html
3. **PostgreSQL 16 Documentation — Aggregate Functions (SUM, etc.)** — https://www.postgresql.org/docs/16/functions-aggregate.html
4. **MySQL 8.0 Reference Manual — Numeric Functions and Operators** — https://dev.mysql.com/doc/refman/8.0/en/numeric-functions.html
5. **MySQL 8.0 Reference Manual — Precision Math** — https://dev.mysql.com/doc/refman/8.0/en/precision-math.html
6. **Microsoft SQL Server — Mathematical Functions (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/mathematical-functions-transact-sql
7. **Microsoft SQL Server — ROUND (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/round-transact-sql
8. **Microsoft SQL Server — RAND (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/functions/rand-transact-sql
9. **Microsoft SQL Server — Data Types (Transact-SQL)** — https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-types-transact-sql
10. **SQL:2016 Standard — ISO/IEC 9075-2:2016 (Foundation)** — https://www.iso.org/standard/63555.html

---

This cheat sheet provides a comprehensive reference for SQL numeric functions across PostgreSQL, MySQL, and SQL Server, with standard SQL notes where applicable. Always consult your specific RDBMS documentation for version-specific behaviour and extensions.