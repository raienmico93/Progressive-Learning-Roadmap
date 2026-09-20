# SQL Numeric Functions: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** SQL numeric functions are built-in scalar functions that accept numeric input and return numeric values, performing mathematical calculations such as absolute value, rounding, exponentiation, and trigonometry within SQL statements.

**Technical Definition:** Per the ANSI/ISO SQL Standard (ISO/IEC 9075), numeric functions are a category of single-row scalar functions that operate on numeric data types and return numeric values. Most numeric functions return values accurate to 38 decimal digits; transcendental functions (COS, EXP, LN, LOG, SIN, SQRT, TAN) are accurate to 36 decimal digits, while inverse transcendental functions (ACOS, ASIN, ATAN, ATAN2) are accurate to 30 decimal digits. These functions can appear in SELECT lists, WHERE clauses, HAVING clauses, and ORDER BY clauses.

**Beginner-Friendly Explanation:** Numeric functions are like the calculator buttons built into your database. Just as a calculator has buttons for square root, absolute value, and rounding, SQL provides functions that do the same thing to the numbers stored in your tables. You can use them to clean data, perform calculations, generate random numbers, or work with geographic coordinates.

### Key Characteristics

- **Scalar functions:** Each numeric function operates on a single row and returns a single value per row.
- **Type-preserving:** Arithmetic functions such as ABS, CEILING, DEGREES, FLOOR, POWER, RADIANS, and SIGN return a value with the same data type as the input value.
- **Vendor variance:** Function names and syntax differ across database engines (e.g., TRUNC vs. TRUNCATE, RAND vs. RANDOM).
- **Precision-aware:** Most functions are accurate to 38 decimal digits for NUMBER values in Oracle, with transcendental functions being slightly less precise.
- **Deterministic vs. non-deterministic:** All mathematical functions except RAND are deterministic functions.

### Prerequisites

- Basic SQL syntax (SELECT, FROM, WHERE)
- Understanding of numeric data types (INT, DECIMAL, FLOAT, NUMBER)
- Familiarity with arithmetic operators (+, -, *, /, %)
- Basic knowledge of radians vs. degrees for trigonometry

### Related Programming Areas

- **Data Cleaning and ETL:** Rounding, truncation, and absolute value functions are used to normalize data during transformation.
- **Financial Calculations:** ROUND, CEILING, FLOOR, and POWER are essential for currency rounding, interest calculations, and amortization.
- **Statistical Analysis:** LOG, LOG10, and EXP are used in statistical modeling and data distribution analysis.
- **Spatial and Geospatial Queries:** SIN, COS, TAN, RADIANS, and DEGREES are used for coordinate transformations and distance calculations.
- **Testing and Simulation:** RAND with seeding enables reproducible random data generation for test suites.
- **Application Development:** Numeric functions are used in reporting, dashboards, and business logic.

---

## Core Concepts / Key Features

### 1. Basic Arithmetic & Absolute Bounds

#### 1.1 Absolute Value: ABS(n)

**Core Definitions:**

- **Core Definition:** The ABS function returns the absolute (positive) value of a numeric expression.
- **Technical Definition:** ABS(n) returns the magnitude of n without its sign. If n is negative, the result is -n; if n is zero or positive, the result is n. ABS returns NULL if the input is NULL.
- **Beginner-Friendly Explanation:** ABS removes the minus sign from a number. ABS(-5) is 5, and ABS(5) is also 5.

**Purposes:**

- To convert negative values to positive for distance or magnitude calculations
- To normalize data by removing directional signs
- To compute error magnitudes in variance analysis

**Syntax Structures and Rules:**

```sql
ABS(numeric_expression)
```

**Component Breakdown:**

- `numeric_expression`: Any expression that evaluates to a numeric data type

**Syntax Rules:**

- Returns the same data type as the input (INT returns INT, DECIMAL returns DECIMAL).
- Returns NULL if the input is NULL.
- On SQLite, if the input is a string or blob that cannot be converted to a numeric value, ABS returns 0.0.
- On SQLite, ABS throws an integer overflow error if X is -9223372036854775808.

**Annotated Code Examples:**

```sql
-- Basic absolute value calculations
SELECT ABS(-17.4) AS positive_value,
       ABS(42) AS already_positive,
       ABS(0) AS zero_value,
       ABS(-0.5) AS half;
```

**Expected Output:**

| positive_value | already_positive | zero_value | half |
|----------------|------------------|------------|------|
| 17.4           | 42               | 0          | 0.5  |

**Explanation:** ABS converts negative values to their positive counterparts while leaving positive values unchanged.

**Real-World Cases:**

- **Finance:** Computing the absolute difference between budgeted and actual spending.
- **Quality control:** Calculating deviation from target measurements.
- **Navigation:** Computing distance traveled regardless of direction.

**References:**

- Microsoft Learn: ABS (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/abs-transact-sql
- PostgreSQL: Mathematical Functions - https://www.postgresql.org/docs/14/functions-math.html
- Oracle: Numeric Functions - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/sql-language-reference.pdf


#### 1.2 Sign Detection: SIGN(n)

**Core Definitions:**

- **Core Definition:** The SIGN function returns -1, 0, or 1 depending on whether the number is negative, zero, or positive.
- **Technical Definition:** SIGN(n) returns +1 if n > 0, 0 if n = 0, and -1 if n < 0. Returns NULL if n is NULL.
- **Beginner-Friendly Explanation:** SIGN tells you the direction of a number: positive, negative, or zero.

**Purposes:**

- To determine the direction of change (increase, decrease, no change)
- To classify values as positive, negative, or zero
- To use as a multiplier in conditional calculations

**Annotated Code Examples:**

```sql
SELECT SIGN(-15) AS negative_sign,
       SIGN(0) AS zero_sign,
       SIGN(42) AS positive_sign,
       SIGN(-0.001) AS tiny_negative;
```

**Expected Output:**

| negative_sign | zero_sign | positive_sign | tiny_negative |
|---------------|-----------|---------------|---------------|
| -1            | 0         | 1             | -1            |

**Real-World Cases:**

- **Stock analysis:** Determining whether a stock price increased (+1), decreased (-1), or stayed flat (0).
- **Financial reporting:** Classifying variances as favorable or unfavorable.
- **Game development:** Determining movement direction.

**References:**

- Microsoft Learn: SIGN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/sign-transact-sql
- MySQL: Numeric Functions - https://dev.mysql.com/doc/refman/9.7/en/numeric-functions.html


#### 1.3 Modulo / Remainder: MOD(dividend, divisor) or %

**Core Definitions:**

- **Core Definition:** The MOD function returns the remainder of a division operation.
- **Technical Definition:** MOD(dividend, divisor) returns the remainder after dividing the dividend by the divisor. In MySQL and PostgreSQL, the `%` operator is also available. The result has the same sign as the dividend in most databases.
- **Beginner-Friendly Explanation:** MOD is the "remainder" button. 10 MOD 3 = 1 because 3 goes into 10 three times with 1 left over.

**Purposes:**

- To check divisibility (e.g., even/odd detection)
- To cycle through a sequence (e.g., round-robin assignment)
- To extract the fractional part of a number

**Annotated Code Examples:**

```sql
-- Modulo operations
SELECT MOD(10, 3) AS remainder_10_3,
       MOD(10, 5) AS exact_division,
       MOD(-10, 3) AS negative_dividend,
       10 % 3 AS percent_operator;
```

**Expected Output:**

| remainder_10_3 | exact_division | negative_dividend | percent_operator |
|----------------|----------------|-------------------|------------------|
| 1              | 0              | -1                | 1                |

**Real-World Cases:**

- **Scheduling:** Assigning tasks to team members in round-robin fashion.
- **Data partitioning:** Distributing rows across partitions.
- **UI development:** Alternating row colors in reports.

**References:**

- MySQL: Numeric Functions (MOD) - https://dev.mysql.com/doc/refman/9.7/en/numeric-functions.html
- PostgreSQL: Mathematical Functions and Operators - https://www.postgresql.org/docs/14/functions-math.html


### 2. Approximation & Precision Control

#### 2.1 Standard Rounding: ROUND(n, decimals)

**Core Definitions:**

- **Core Definition:** ROUND rounds a number to a specified number of decimal places.
- **Technical Definition:** ROUND(n, d) rounds n to d decimal places. If d is omitted, n is rounded to 0 decimal places (the nearest integer). The rounding method is "round half away from zero" in most databases.
- **Beginner-Friendly Explanation:** ROUND is the standard rounding button. ROUND(3.14159, 2) gives 3.14.

**Purposes:**

- To format numeric output for display
- To comply with currency rounding rules
- To reduce precision for reporting and analysis

**Annotated Code Examples:**

```sql
SELECT ROUND(3.14159, 2) AS rounded_pi,
       ROUND(2.5) AS round_half_up,
       ROUND(3.5) AS round_half_up_2,
       ROUND(-2.5) AS negative_rounding,
       ROUND(1234.5678, -2) AS negative_decimals;
```

**Expected Output:**

| rounded_pi | round_half_up | round_half_up_2 | negative_rounding | negative_decimals |
|------------|---------------|-----------------|-------------------|-------------------|
| 3.14       | 3             | 4               | -3                | 1200              |

**Explanation:** ROUND uses "round half away from zero," so 2.5 rounds to 3 and -2.5 rounds to -3. Negative decimal places round to the left of the decimal point.

**Real-World Cases:**

- **Finance:** Rounding currency amounts to 2 decimal places.
- **Reporting:** Presenting percentages with 1 decimal place.
- **Engineering:** Rounding measurements to the nearest unit.

**References:**

- Microsoft Learn: ROUND (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/round-transact-sql
- PostgreSQL: Mathematical Functions (ROUND) - https://www.postgresql.org/docs/14/functions-math.html


#### 2.2 Truncation: TRUNC(n, decimals) / TRUNCATE()

**Core Definitions:**

- **Core Definition:** TRUNC (Oracle, PostgreSQL) or TRUNCATE (MySQL) chops off decimal places without rounding.
- **Technical Definition:** TRUNC(n, d) returns n truncated to d decimal places. Unlike ROUND, TRUNC always rounds toward zero (truncates) rather than rounding to the nearest value.
- **Beginner-Friendly Explanation:** TRUNC just cuts off the extra digits. TRUNC(3.14159, 2) gives 3.14, and TRUNC(3.999, 2) also gives 3.99 (not 4.00).

**Purposes:**

- To remove unwanted decimal places without rounding
- To implement "floor toward zero" behavior
- To extract the integer part of a number

**Annotated Code Examples:**

```sql
-- MySQL: TRUNCATE
SELECT TRUNCATE(3.14159, 2) AS truncated_pi,
       TRUNCATE(3.999, 2) AS no_rounding,
       TRUNCATE(123.456, 0) AS truncated_integer;
```

**Expected Output:**

| truncated_pi | no_rounding | truncated_integer |
|--------------|-------------|-------------------|
| 3.14         | 3.99        | 123               |

**Real-World Cases:**

- **Data analysis:** Removing insignificant decimal places without affecting the value.
- **Gaming:** Calculating experience points where partial points are discarded.
- **Financial systems:** Truncating calculated interest to the cent (banker's truncation).

**References:**

- MySQL: Numeric Functions (TRUNCATE) - https://dev.mysql.com/doc/refman/9.7/en/numeric-functions.html
- Oracle: TRUNC (number) - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/TRUNC.html


#### 2.3 Ceiling: CEIL(n) / CEILING(n)

**Core Definitions:**

- **Core Definition:** CEIL (or CEILING) returns the smallest integer greater than or equal to the input value.
- **Technical Definition:** CEIL(n) rounds n up to the nearest integer. For negative numbers, CEIL(-42.8) returns -42 because -42 is greater than -42.8.
- **Beginner-Friendly Explanation:** CEIL always rounds up. CEIL(4.1) is 5, and CEIL(-4.9) is -4.

**Annotated Code Examples:**

```sql
SELECT CEIL(42.2) AS round_up,
       CEIL(-42.8) AS negative_round_up,
       CEILING(0.5) AS half_up,
       CEIL(5.0) AS exact_integer;
```

**Expected Output:**

| round_up | negative_round_up | half_up | exact_integer |
|----------|-------------------|---------|---------------|
| 43       | -42               | 1       | 5             |

**Explanation:** CEIL(42.2) → 43; CEIL(-42.8) → -42 (because -42 > -42.8).

**Real-World Cases:**

- **Inventory:** Calculating the number of boxes needed to pack items (always round up).
- **Pagination:** Computing the number of pages for a result set.
- **Billing:** Rounding up partial hours to the next full hour.

**References:**

- PostgreSQL: Mathematical Functions (CEIL) - https://www.postgresql.org/docs/14/functions-math.html
- Microsoft Learn: CEILING (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/ceiling-transact-sql


#### 2.4 Floor: FLOOR(n)

**Core Definitions:**

- **Core Definition:** FLOOR returns the largest integer less than or equal to the input value.
- **Technical Definition:** FLOOR(n) rounds n down to the nearest integer. FLOOR(-42.2) returns -43 because -43 is less than -42.2.
- **Beginner-Friendly Explanation:** FLOOR always rounds down. FLOOR(4.9) is 4, and FLOOR(-4.1) is -5.

**Annotated Code Examples:**

```sql
SELECT FLOOR(42.8) AS round_down,
       FLOOR(-42.2) AS negative_round_down,
       FLOOR(0.9) AS almost_one,
       FLOOR(5.0) AS exact_integer;
```

**Expected Output:**

| round_down | negative_round_down | almost_one | exact_integer |
|------------|---------------------|------------|---------------|
| 42         | -43                 | 0          | 5             |

**Real-World Cases:**

- **Age calculation:** Computing age from date of birth (floor of years).
- **Budgeting:** Determining how many items can be purchased with a fixed budget.
- **Time tracking:** Counting complete hours worked.

**References:**

- PostgreSQL: Mathematical Functions (FLOOR) - https://www.postgresql.org/docs/14/functions-math.html
- Microsoft Learn: FLOOR (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/floor-transact-sql


### 3. Exponential, Root, & Logarithmic Scales

#### 3.1 Powers and Square Roots: POWER(base, exponent) and SQRT(n)

**Core Definitions:**

- **Core Definition:** POWER raises a base to a specified exponent; SQRT returns the square root of a number.
- **Technical Definition:** POWER(a, b) returns a raised to the power of b (a^b). SQRT(n) returns the square root of n (n must be non-negative). PostgreSQL also supports the `^` operator for exponentiation and the `|/` operator for square root.
- **Beginner-Friendly Explanation:** POWER is the exponent button (x^y). SQRT is the square root button.

**Annotated Code Examples:**

```sql
SELECT POWER(2, 10) AS two_to_the_tenth,
       POWER(9, 0.5) AS square_root_via_power,
       SQRT(144) AS square_root,
       POWER(2, -2) AS negative_exponent;
```

**Expected Output:**

| two_to_the_tenth | square_root_via_power | square_root | negative_exponent |
|------------------|----------------------|-------------|-------------------|
| 1024             | 3                    | 12          | 0.25              |

**Real-World Cases:**

- **Compound interest:** POWER(1 + rate, years) for future value calculations.
- **Physics:** Computing kinetic energy (½mv²) or gravitational force.
- **Geometry:** Computing distances using the Pythagorean theorem.

**References:**

- PostgreSQL: Mathematical Functions (POWER, SQRT) - https://www.postgresql.org/docs/14/functions-math.html
- MySQL: Numeric Functions (POW, POWER) - https://dev.mysql.com/doc/refman/9.7/en/numeric-functions.html


#### 3.2 Natural Logarithms: LN(n) or LOG(n)

**Core Definitions:**

- **Core Definition:** LN (and LOG in some databases) returns the natural logarithm (base e) of a number.
- **Technical Definition:** LN(n) returns the power to which e (≈2.71828) must be raised to equal n. In PostgreSQL, LOG(n) is also the natural logarithm. In MySQL, LOG(n) is the natural logarithm, and LOG(B, X) returns the logarithm of X to base B.
- **Beginner-Friendly Explanation:** LN is the "reverse of EXP." If EXP(2) = 7.389, then LN(7.389) = 2.

**Annotated Code Examples:**

```sql
SELECT LN(1) AS ln_one,
       LN(2.718281828) AS ln_e,
       LN(10) AS ln_ten;
```

**Expected Output:**

| ln_one | ln_e | ln_ten |
|--------|------|--------|
| 0      | 1    | 2.302585 |

**Real-World Cases:**

- **Machine learning:** Computing log-likelihood or cross-entropy loss.
- **Finance:** Calculating continuously compounded returns.
- **Data science:** Transforming skewed data for normal distribution.

**References:**

- MySQL: Numeric Functions (LN, LOG) - https://dev.mysql.com/doc/refman/9.7/en/numeric-functions.html
- PostgreSQL: Mathematical Functions (LN) - https://www.postgresql.org/docs/14/functions-math.html


#### 3.3 Base-10 Logarithms: LOG10(n) or LOG(10, n)

**Core Definitions:**

- **Core Definition:** LOG10 (or LOG(10, n)) returns the base-10 logarithm of a number.
- **Technical Definition:** LOG10(n) returns the power to which 10 must be raised to equal n. Used heavily in statistical data distribution modeling and scientific calculations.
- **Beginner-Friendly Explanation:** LOG10 tells you how many zeros are in a power of 10. LOG10(100) = 2, LOG10(1000) = 3.

**Annotated Code Examples:**

```sql
SELECT LOG10(10) AS log_10,
       LOG10(100) AS log_100,
       LOG10(1) AS log_1,
       LOG(10, 1000) AS log_base_10_via_log;
```

**Expected Output:**

| log_10 | log_100 | log_1 | log_base_10_via_log |
|--------|---------|-------|---------------------|
| 1      | 2       | 0     | 3                   |

**Real-World Cases:**

- **Earthquake measurement:** Richter scale uses base-10 logarithms.
- **Sound intensity:** Decibel calculations use LOG10.
- **Data visualization:** Log-scale charts for exponential data.

**References:**

- MySQL: Numeric Functions (LOG10) - https://dev.mysql.com/doc/refman/9.7/en/numeric-functions.html
- PostgreSQL: Mathematical Functions (LOG) - https://www.postgresql.org/docs/14/functions-math.html


#### 3.4 Exponential: EXP(n)

**Core Definitions:**

- **Core Definition:** EXP(n) returns e raised to the power of n (e^n).
- **Technical Definition:** EXP(n) computes the exponential function, where e is the base of natural logarithms (≈2.718281828). It is the inverse of LN.
- **Beginner-Friendly Explanation:** EXP is the "e to the power of" button. EXP(1) ≈ 2.718.

**Annotated Code Examples:**

```sql
SELECT EXP(0) AS exp_zero,
       EXP(1) AS exp_one,
       EXP(2) AS exp_two,
       LN(EXP(5)) AS round_trip;
```

**Expected Output:**

| exp_zero | exp_one | exp_two | round_trip |
|----------|---------|---------|------------|
| 1        | 2.718   | 7.389   | 5          |

**Real-World Cases:**

- **Population growth:** Modeling exponential growth or decay.
- **Radioactive decay:** Computing remaining substance over time.
- **Finance:** Continuous compounding interest calculations.

**References:**

- PostgreSQL: Mathematical Functions (EXP) - https://www.postgresql.org/docs/14/functions-math.html
- MySQL: Numeric Functions (EXP) - https://dev.mysql.com/doc/refman/9.7/en/numeric-functions.html


### 4. Randomness & Statistical Generation

#### 4.1 Random Float Generation: RAND() / RANDOM()

**Core Definitions:**

- **Core Definition:** RAND (SQL Server, MySQL) or RANDOM (PostgreSQL, SQLite) returns a random floating-point value between 0 and 1.
- **Technical Definition:** In SQL Server, RAND() returns a pseudo-random float value from 0 through 1, exclusive. PostgreSQL's RANDOM() returns a random value in the range 0.0 <= x < 1.0. SQLite's RANDOM() returns a random 64-bit signed integer, not a float between 0 and 1.
- **Beginner-Friendly Explanation:** RAND gives you a random number between 0 and 1. Multiply it by 100 to get a random percentage, or by 10 to get a random number between 0 and 10.

**Annotated Code Examples:**

```sql
-- SQL Server / MySQL: Generate random numbers
SELECT RAND() AS random_float,
       ROUND(RAND() * 100, 2) AS random_percentage,
       FLOOR(RAND() * 10)     AS random_0_to_9,
       ROUND(RAND() * 10, 0)  AS random_0_to_10;
```

**Expected Output (varies each run):**

| random_float | random_percentage | random_0_to_9 | random_0_to_10 |
|--------------|-------------------|---------------|----------------|
| 0.7245       | 72.45             | 7             | 7              |

**Real-World Cases:**

- **Testing:** Generating random test data.
- **Sampling:** Selecting a random sample of rows.
- **Gaming:** Randomizing loot drops or enemy behavior.

**References:**

- Microsoft Learn: RAND (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/rand-transact-sql
- PostgreSQL: Mathematical Functions (RANDOM) - https://www.postgresql.org/docs/14/functions-math.html


#### 4.2 Deterministic Seeding: RAND(seed)

**Core Definitions:**

- **Core Definition:** Passing a seed value to RAND produces a reproducible sequence of "random" numbers.
- **Technical Definition:** When RAND(seed) is called with a specified seed value, repetitive calls with the same seed return the same results. For one connection, if RAND() is called with a specified seed value, all subsequent calls produce results based on the seeded call.
- **Beginner-Friendly Explanation:** Seeding makes random numbers repeatable. RAND(5) will always give the same sequence, which is useful for testing.

**Purposes:**

- To generate reproducible test data
- To ensure consistent results across test runs
- To debug random-dependent logic

**Annotated Code Examples:**

```sql
-- SQL Server: Seeded random numbers (reproducible)
SELECT RAND(100) AS seeded_1,
       RAND() AS next_from_seed,
       RAND() AS next_from_seed_2;
```

**Expected Output (consistent across runs):**

| seeded_1 | next_from_seed | next_from_seed_2 |
|----------|----------------|------------------|
| 0.xxx    | 0.yyy          | 0.zzz            |

**Explanation:** The same seed produces the same sequence of numbers every time, enabling reproducible tests.

**Important Note:** When RAND() is used in an UPDATE or INSERT query, all affected rows get the same value.

**Real-World Cases:**

- **Unit testing:** Generating predictable random test data.
- **Simulations:** Reproducing simulation scenarios.
- **A/B testing:** Ensuring consistent randomization across groups.

**References:**

- Microsoft Learn: RAND (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/rand-transact-sql


### 5. Trigonometry & Geometric Math (Spatial Queries)

#### 5.1 Core Trig Ratios: SIN(n), COS(n), TAN(n)

**Core Definitions:**

- **Core Definition:** SIN, COS, and TAN return the sine, cosine, and tangent of an angle, respectively.
- **Technical Definition:** All trigonometric functions take arguments and return values of type double precision. Trigonometric function arguments are expressed in radians.
- **Beginner-Friendly Explanation:** These are the basic trigonometry functions. They expect angles in radians, not degrees.

**Annotated Code Examples:**

```sql
SELECT SIN(0) AS sin_zero,
       COS(0) AS cos_zero,
       TAN(0) AS tan_zero,
       SIN(PI()/2) AS sin_90_degrees;
```

**Expected Output:**

| sin_zero | cos_zero | tan_zero | sin_90_degrees |
|----------|----------|----------|----------------|
| 0        | 1        | 0        | 1              |

**Real-World Cases:**

- **GIS:** Computing distances between geographic coordinates.
- **Physics:** Modeling wave behavior and oscillations.
- **Game development:** Calculating projectile trajectories.

**References:**

- PostgreSQL: Mathematical Functions (Trig) - https://www.postgresql.org/docs/14/functions-math.html
- Microsoft Learn: SIN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/sin-transact-sql


#### 5.2 Inverse Trig: ASIN(n), ACOS(n), ATAN(n)

**Core Definitions:**

- **Core Definition:** ASIN, ACOS, and ATAN return the inverse trigonometric functions, recovering an angle from a trigonometric ratio.
- **Technical Definition:** Inverse functions return values expressed in radians. ASIN and ACOS return values in the range [-π/2, π/2] and [0, π], respectively. ATAN2(y, x) returns the angle whose tangent is y/x, using the signs of both arguments to determine the quadrant.
- **Beginner-Friendly Explanation:** If SIN(30°) = 0.5, then ASIN(0.5) = 30° (in radians).

**Annotated Code Examples:**

```sql
SELECT ASIN(1) AS asin_one,
       ACOS(0) AS acos_zero,
       ATAN(1) AS atan_one,
       ATAN2(1, 1) AS atan2_one_one;
```

**Expected Output:**

| asin_one | acos_zero | atan_one | atan2_one_one |
|----------|-----------|----------|---------------|
| 1.5708   | 1.5708    | 0.7854   | 0.7854        |

**Real-World Cases:**

- **Navigation:** Computing bearings from coordinate differences.
- **Robotics:** Calculating joint angles from sensor data.
- **Astronomy:** Determining celestial object positions.

**References:**

- PostgreSQL: Mathematical Functions (Inverse Trig) - https://www.postgresql.org/docs/14/functions-math.html
- Oracle: Numeric Functions (ASIN, ACOS, ATAN) - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/sql-language-reference.pdf


#### 5.3 Angular Conversions: RADIANS(degrees) and DEGREES(radians)

**Core Definitions:**

- **Core Definition:** RADIANS converts degrees to radians; DEGREES converts radians to degrees.
- **Technical Definition:** RADIANS(d) returns d * π / 180. DEGREES(r) returns r * 180 / π. These functions are essential for spatial queries because trigonometric functions expect radians, but coordinates are often expressed in degrees.
- **Beginner-Friendly Explanation:** RADIANS and DEGREES are the "unit converters" for angles. RADIANS(180) = π ≈ 3.14159.

**Annotated Code Examples:**

```sql
SELECT RADIANS(180) AS pi_radians,
       DEGREES(PI()) AS one_eighty,
       RADIANS(90) AS half_pi,
       DEGREES(PI()/2) AS ninety_degrees;
```

**Expected Output:**

| pi_radians | one_eighty | half_pi | ninety_degrees |
|------------|------------|---------|----------------|
| 3.14159    | 180        | 1.5708  | 90             |

**Real-World Cases:**

- **GIS:** Converting GPS coordinates (degrees) to radians for distance calculations.
- **Mapping:** Transforming between different coordinate systems.
- **Surveying:** Computing angles from field measurements.

**References:**

- MySQL: Numeric Functions (RADIANS, DEGREES) - https://dev.mysql.com/doc/refman/9.7/en/numeric-functions.html
- Microsoft Learn: RADIANS (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/radians-transact-sql


## Summary Tables

### Numeric Function Availability Across Databases

| Function | PostgreSQL | MySQL | SQL Server | Oracle | SQLite |
|----------|-----------|-------|------------|--------|--------|
| ABS | ✓ | ✓ | ✓ | ✓ | ✓ |
| SIGN | ✓ | ✓ | ✓ | ✓ | ✗ |
| MOD / % | ✓ | ✓ | ✓ (%) | ✓ (MOD) | ✓ (%), MOD |
| ROUND | ✓ | ✓ | ✓ | ✓ | ✓ |
| TRUNC / TRUNCATE | ✓ | ✓ | ✗ | ✓ | ✗ |
| CEIL / CEILING | ✓ | ✓ | ✓ | ✓ | ✗ |
| FLOOR | ✓ | ✓ | ✓ | ✓ | ✗ |
| POWER / POW | ✓ | ✓ | ✓ | ✓ | ✗ |
| SQRT | ✓ | ✓ | ✓ | ✓ | ✗ |
| LN | ✓ | ✓ | ✓ (LOG) | ✓ | ✗ |
| LOG10 | ✓ (LOG) | ✓ | ✓ (LOG10) | ✓ (LOG) | ✗ |
| EXP | ✓ | ✓ | ✓ | ✓ | ✗ |
| RAND / RANDOM | ✓ (RANDOM) | ✓ (RAND) | ✓ (RAND) | ✓ (DBMS_RANDOM) | ✓ (RANDOM) |
| SIN, COS, TAN | ✓ | ✓ | ✓ | ✓ | ✗ |
| ASIN, ACOS, ATAN | ✓ | ✓ | ✓ | ✓ | ✗ |
| RADIANS, DEGREES | ✓ | ✓ | ✓ | ✓ | ✗ |

### Rounding Function Comparison

| Function | Behavior | Example | Result |
|----------|----------|---------|--------|
| ROUND(3.5) | Round half away from zero | ROUND(3.5) | 4 |
| ROUND(-3.5) | Round half away from zero | ROUND(-3.5) | -4 |
| TRUNC(3.9) | Chop toward zero | TRUNC(3.9) | 3 |
| TRUNC(-3.9) | Chop toward zero | TRUNC(-3.9) | -3 |
| CEIL(3.1) | Round up | CEIL(3.1) | 4 |
| CEIL(-3.9) | Round up (toward +∞) | CEIL(-3.9) | -3 |
| FLOOR(3.9) | Round down | FLOOR(3.9) | 3 |
| FLOOR(-3.1) | Round down (toward -∞) | FLOOR(-3.1) | -4 |

### Random Function Comparison

| Database | Function | Range | Seeding |
|----------|----------|-------|---------|
| SQL Server | RAND() | 0 ≤ x < 1 | RAND(seed) |
| MySQL | RAND() | 0 ≤ x < 1 | RAND(seed) |
| PostgreSQL | RANDOM() | 0 ≤ x < 1 | setseed() |
| SQLite | RANDOM() | Full 64-bit integer | Not supported |

---

## References

- PostgreSQL: Mathematical Functions and Operators - https://www.postgresql.org/docs/14/functions-math.html
- PostgreSQL: Math Functions (current) - https://www.postgresql.org/docs/current/functions-math.html
- MySQL: Numeric Functions and Operators - https://dev.mysql.com/doc/refman/9.7/en/numeric-functions.html
- Microsoft Learn: Mathematical Functions (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/mathematical-functions-transact-sql
- Microsoft Learn: RAND (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/rand-transact-sql
- Microsoft Learn: ABS (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/abs-transact-sql
- Microsoft Learn: SIGN (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/sign-transact-sql
- Microsoft Learn: CEILING (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/ceiling-transact-sql
- Microsoft Learn: FLOOR (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/floor-transact-sql
- Microsoft Learn: ROUND (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/round-transact-sql
- Microsoft Learn: RADIANS (Transact-SQL) - https://learn.microsoft.com/en-us/sql/t-sql/functions/radians-transact-sql
- Oracle: SQL Language Reference (Numeric Functions) - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/sql-language-reference.pdf
- Oracle: TRUNC (number) - https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/TRUNC.html
- SQLite: Built-In Scalar SQL Functions - https://wasm-testing.sqlite.org/draft/lang_corefunc.html