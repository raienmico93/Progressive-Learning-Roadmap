# Pandas Numeric Operations: A Comprehensive Programming Cheat Sheet

---

## Topic Overview

### Definitions

**Core Definition:** Numeric operations in pandas are vectorized, element-wise computations performed on entire Series or DataFrame objects using standard mathematical operators (+, -, *, /, %, **) and their method-based equivalents (`.add()`, `.sub()`, `.mul()`, `.div()`), with automatic index alignment.

**Technical Definition:** Pandas leverages NumPy's universal functions (ufuncs) to perform vectorized arithmetic on labeled data structures. Binary operations between Series or DataFrames automatically align on index and column labels, introducing `NaN` for non-matching labels unless a `fill_value` is specified. The flexible arithmetic methods (`.add()`, `.sub()`, `.mul()`, `.div()`, `.mod()`, `.pow()`) expose additional parameters (`axis`, `level`, `fill_value`) for precise control over alignment and broadcasting. Mathematical transformations such as `abs()`, `np.sqrt()`, `np.exp()`, and `np.log()` are applied element-wise. Cumulative operations (`.cumsum()`, `.cumprod()`, `.cummin()`, `.cummax()`) compute running aggregates along an axis, preserving the location of `NaN` values by default.

**Beginner-Friendly Explanation:** Pandas lets you do math on entire columns at once, rather than looping through rows one by one. You can add two columns together, multiply by a constant, take the square root of every value, or compute a running total—all with simple operators and methods. When you combine two DataFrames, pandas lines them up by their row and column labels, so you don't have to worry about order.

### Key Characteristics

- **Vectorized execution:** Arithmetic operations run at C-level speed via NumPy ufuncs, eliminating Python loops.
- **Automatic index alignment:** Binary operations align on index and column labels, filling non-matching entries with `NaN` by default.
- **Flexible arithmetic methods:** `.add()`, `.sub()`, `.mul()`, `.div()`, `.mod()`, `.pow()` provide `fill_value` and `axis` parameters for handling mismatched labels.
- **Division by zero:** Pandas returns `[-inf, nan, inf]` for division by zero, differing from NumPy's convention in some dtype combinations.
- **Cumulative methods preserve NaN locations:** `.cumsum()` and `.cumprod()` ignore `NaN` values by default but preserve their positions in the output.

### Prerequisites

- Basic Python knowledge (variables, operators, functions).
- Familiarity with pandas Series and DataFrame objects.
- Understanding of NumPy arrays and broadcasting.
- Installation of pandas (`pip install pandas`).

### Related Programming Areas

- **Data Analysis:** Computing ratios, differences, and growth rates across columns.
- **Feature Engineering:** Creating derived features via arithmetic and mathematical transforms.
- **Time Series Analysis:** Cumulative returns, running totals, and exponential smoothing.
- **Financial Modeling:** Compound interest, log returns, and percentage changes.

### Core Concepts / Features

1. Vectorized Arithmetic: Element-wise Math Operators
2. Index Alignment: Automatic Label-Based Computation
3. Vectorized Division Safety: Division by Zero and Infinity
4. Absolute Values and Transformations: abs(), exp(), log(), sqrt()
5. Cumulative Variations: Running Trends with cumsum(), cumprod(), cummin(), cummax()


## 1. Vectorized Arithmetic: Element-wise Transformations

### Definitions

**Core Definition:** Vectorized arithmetic applies standard mathematical operators (+, -, *, /, %, **) to entire Series or DataFrames in a single operation, computing each element individually without explicit Python loops.

**Technical Definition:** Pandas overrides Python's arithmetic operators to use NumPy's vectorized ufuncs. For Series and DataFrames, the operators `+`, `-`, `*`, `/`, `//`, `%`, and `**` perform element-wise operations. Scalar operands are broadcast across all elements. The equivalent method-based forms (`.add()`, `.sub()`, `.mul()`, `.div()`, `.floordiv()`, `.mod()`, `.pow()`) provide additional parameters for alignment control.

**Beginner-Friendly Explanation:** Instead of writing `for i in range(len(df)): df['C'][i] = df['A'][i] + df['B'][i]`, you just write `df['C'] = df['A'] + df['B']`. Pandas does the loop for you in optimized code, which is much faster.

### Purposes

- To perform element-wise arithmetic on entire columns without explicit loops.
- To broadcast scalar values across all elements of a Series or DataFrame.
- To combine multiple columns using standard mathematical operators.
- To leverage NumPy's optimized C loops for maximum performance.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Operator form
df['C'] = df['A'] + df['B']
df['C'] = df['A'] - df['B']
df['C'] = df['A'] * df['B']
df['C'] = df['A'] / df['B']
df['C'] = df['A'] % df['B']
df['C'] = df['A'] ** df['B']

# Method form
df['C'] = df['A'].add(df['B'])
df['C'] = df['A'].sub(df['B'])
df['C'] = df['A'].mul(df['B'])
df['C'] = df['A'].div(df['B'])
df['C'] = df['A'].mod(df['B'])
df['C'] = df['A'].pow(df['B'])
```

**Component Breakdown:**

| Operator | Method | Description |
|----------|--------|-------------|
| `+` | `.add()` | Addition |
| `-` | `.sub()` | Subtraction |
| `*` | `.mul()` | Multiplication |
| `/` | `.div()` | Float division |
| `//` | `.floordiv()` | Integer division |
| `%` | `.mod()` | Modulo (remainder) |
| `**` | `.pow()` | Exponentiation |

**Syntax Rules:**

- All operators work element-wise on Series and DataFrames.
- Scalars are broadcast to every element.
- When combining Series with different indexes, pandas aligns on labels and introduces `NaN` for non-matching labels.
- The `.add()` method with `fill_value` replaces non-matching entries with the specified value instead of `NaN`.

**Constraints and Limitations:**

- Mismatched indexes produce `NaN` values; use `fill_value` to handle them.
- Operations between DataFrames with different column sets align on column labels.
- Division by zero produces `inf`, `-inf`, or `NaN`, not an error.

### Annotated Code Examples

**Example 1: Basic Arithmetic Operations**

```python
import pandas as pd

df = pd.DataFrame({
    'A': [10, 20, 30],
    'B': [3, 4, 5]
})

# Addition, subtraction, multiplication, division
df['sum'] = df['A'] + df['B']
df['diff'] = df['A'] - df['B']
df['product'] = df['A'] * df['B']
df['ratio'] = df['A'] / df['B']
print(df)
```

Expected output:
```
    A  B  sum  diff  product  ratio
0  10  3   13     7       30   3.33
1  20  4   24    16       80   5.00
2  30  5   35    25      150   6.00
```

**Why this output:** Each operator performs element-wise computation between corresponding elements of columns A and B. The result is assigned to a new column.

**Example 2: Scalar Operations and Broadcasting**

```python
# Scalar addition and multiplication
df['A_plus_10'] = df['A'] + 10
df['A_squared'] = df['A'] ** 2
print(df[['A', 'A_plus_10', 'A_squared']])
```

Expected output:
```
    A  A_plus_10  A_squared
0  10         20        100
1  20         30        400
2  30         40        900
```

**Why this output:** The scalar `10` is broadcast across every element of column A. The exponentiation operator `**` squares each element.

**Example 3: Modulo and Floor Division**

```python
# Modulo (remainder) and floor division
df['A_mod_3'] = df['A'] % 3
df['A_floor_div_3'] = df['A'] // 3
print(df[['A', 'A_mod_3', 'A_floor_div_3']])
```

Expected output:
```
#     A  A_mod_3  A_floor_div_3
# 0  10        1              3
# 1  20        2              6
# 2  30        0             10
```

**Why this output:** `%` returns the remainder after division (10 % 3 = 1). `//` returns the integer quotient (10 // 3 = 3).

### Real-World Cases

- **Financial analysis:** Compute revenue as `price * quantity` across a sales DataFrame.
- **Unit conversion:** Convert temperatures with `df['temp_f'] = df['temp_c'] * 9/5 + 32`.
- **Data normalization:** Standardize columns using `(df - df.mean()) / df.std()`.

### References

- pandas.DataFrame.add — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.add.html
- pandas.DataFrame.mul — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.mul.html
- pandas.DataFrame.div — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.div.html
- pandas.DataFrame.pow — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.pow.html
- Binary operations (User Guide) — https://pandas.pydata.org/docs/user_guide/basics.html#binary-operation


## 2. Index Alignment: Automatic Label-Based Computation

### Definitions

**Core Definition:** Index alignment is pandas' automatic process of matching elements by their index and column labels during binary operations, rather than by position.

**Technical Definition:** When performing a binary operation between two Series or DataFrames, pandas aligns the operands on their index labels (and column labels for DataFrames). The resulting index is the **union** of the two indexes. Entries present in one operand but not the other produce `NaN` in the result. The flexible arithmetic methods accept a `fill_value` parameter that substitutes a value for non-matching entries before computation, and an `axis` parameter to control broadcasting direction.

**Beginner-Friendly Explanation:** If you add two DataFrames, pandas doesn't just add row 1 to row 1. It looks at the labels—if both have a row labeled “A”, it adds those. If one has a row “B” that the other doesn't, the result gets `NaN` for that row. You can use `fill_value` to treat missing rows as zeros instead.

### Purposes

- To combine data from different sources while respecting label-based correspondence.
- To handle mismatched indexes gracefully with `NaN` or a custom `fill_value`.
- To broadcast a Series across a DataFrame's columns or rows using the `axis` parameter.
- To align data before performing arithmetic that requires matched labels.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Automatic alignment (NaN for non-matching labels)
result = df1 + df2

# Fill non-matching labels with a value before computation
result = df1.add(df2, fill_value=0)

# Broadcast a Series across columns (axis=1)
result = df.add(series, axis=1)

# Broadcast a Series across rows (axis=0)
result = df.add(series, axis=0)
```

**Component Breakdown:**

| Parameter | Description |
|-----------|-------------|
| `other` | Scalar, sequence, Series, or DataFrame |
| `axis` | `0` or `'index'`: match Series index with DataFrame index; `1` or `'columns'`: match with columns |
| `level` | Broadcast across a level of a MultiIndex |
| `fill_value` | Value to substitute for non-matching labels before computation |

**Syntax Rules:**

- When `fill_value=None` (default), non-matching labels produce `NaN`.
- `fill_value` only fills missing values from **one** of the operands; if both are missing, the result is still `NaN`.
- The `axis` parameter determines how a Series is aligned against a DataFrame.
- For DataFrames, the resulting index is the union of both indexes.

**Constraints and Limitations:**

- `fill_value` does not fill existing `NaN` values in the data; it only fills entries introduced by index misalignment.
- If data in both corresponding locations is missing, the result remains missing.
- Alignment can produce large results when indexes are very different.

### Annotated Code Examples

**Example 1: Automatic Alignment with NaN**

```python
import pandas as pd

df1 = pd.DataFrame({
    'A': [1, 2, 3],
    'B': [4, 5, 6]
}, index=['x', 'y', 'z'])

df2 = pd.DataFrame({
    'A': [10, 20],
    'B': [40, 50]
}, index=['y', 'z'])

# Automatic alignment produces NaN for non-matching row 'x'
result = df1 + df2
print(result)
# Expected output:
#       A     B
# x   NaN   NaN
# y  12.0  45.0
# z  23.0  56.0
```

**Why this output:** Row `'x'` exists only in `df1`, so the result has `NaN` for that row. Rows `'y'` and `'z'` exist in both DataFrames, so their values are added.

**Example 2: Using fill_value to Handle Mismatched Labels**

```python
# fill_value=0 treats missing labels as zero
result_filled = df1.add(df2, fill_value=0)
print(result_filled)
# Expected output:
#       A     B
# x   1.0   4.0
# y  12.0  45.0
# z  23.0  56.0
```

**Why this output:** With `fill_value=0`, row `'x'` from `df1` is treated as `0` in `df2` (missing), so the result is `1 + 0 = 1` and `4 + 0 = 4`.

**Example 3: Broadcasting a Series Across Columns**

```python
# Broadcast a Series across DataFrame columns
adjustment = pd.Series({'A': 100, 'B': 200})
result_broadcast = df1.add(adjustment, axis=1)
print(result_broadcast)
# Expected output:
#      A    B
# x  101  204
# y  102  205
# z  103  206
```

**Why this output:** With `axis=1`, the Series `adjustment` is aligned with the DataFrame's columns. The value 100 is added to every element in column A, and 200 to every element in column B.

### Real-World Cases

- **Multi-source data integration:** Combine sales data from different regions where some regions are missing in one source.
- **Time series alignment:** Add two time series with different date ranges, using `fill_value=0` for missing dates.
- **Portfolio calculations:** Combine holdings from different accounts, treating missing holdings as zero.

### References

- pandas.DataFrame.add (fill_value parameter) — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.add.html
- Index alignment (User Guide) — https://pandas.pydata.org/docs/user_guide/basics.html#index-alignment
- pandas.DataFrame.align — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.align.html


## 3. Vectorized Division Safety: Handling Division by Zero

### Definitions

**Core Definition:** Division by zero in pandas produces `inf`, `-inf`, or `NaN` rather than raising an error, following IEEE 754 floating-point conventions with pandas-specific behaviour for different dtype combinations.

**Technical Definition:** Pandas conventions regarding division by zero differ from NumPy in specific ways. For float dtypes, dividing a non-zero number by zero yields `inf` or `-inf` depending on the sign of the numerator; dividing zero by zero yields `NaN`. For integer dtypes, pandas returns `[-inf, nan, inf]` for all dtype combinations, whereas NumPy may return different results for integer division. The resulting infinity values can be replaced using `.replace()` or masked using `.where()`.

**Beginner-Friendly Explanation:** If you divide a number by zero, pandas doesn't crash—it gives you “infinity” (`inf`). If you divide zero by zero, it gives you `NaN` (not a number). You can replace these special values with something more useful, like zero, if your analysis requires it.

### Purposes

- To understand the default behaviour when division by zero occurs in a dataset.
- To detect and replace `inf` and `-inf` values with meaningful substitutes (e.g., 0 or the column mean).
- To avoid errors in downstream calculations that cannot handle infinity.
- To implement safe division logic that handles zero denominators gracefully.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Division that may produce inf
result = df['A'] / df['B']

# Replace inf and -inf with a value
result = result.replace([np.inf, -np.inf], 0)

# Replace inf with NaN, then fill
result = result.replace([np.inf, -np.inf], np.nan).fillna(0)

# Safe division using numpy
import numpy as np
result = np.divide(df['A'], df['B'], where=df['B'] != 0, out=np.zeros_like(df['A'], dtype=float))
```

**Component Breakdown:**

| Operation | Result |
|-----------|--------|
| `non_zero / 0` | `inf` or `-inf` |
| `0 / 0` | `NaN` |
| `non_zero / 0.0` | `inf` or `-inf` |
| `0 / 0.0` | `NaN` |

**Syntax Rules:**

- Division by zero does not raise an exception; it returns `inf`, `-inf`, or `NaN`.
- `.replace([np.inf, -np.inf], value)` replaces infinity values.
- `np.divide(..., where=..., out=...)` provides a safe division pattern.
- To handle both `inf` and `-inf` in one call, pass a list to `.replace()`.

**Constraints and Limitations:**

- `inf` values can propagate through subsequent calculations.
- Comparisons involving `inf` (e.g., `inf > 1000`) return `True`, which may be unexpected.
- `.replace()` with `inf` requires `np.inf` or `float('inf')`, not the string `'inf'`.

### Annotated Code Examples

**Example 1: Division by Zero Produces inf**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'numerator': [10, -10, 0, 5],
    'denominator': [2, 0, 0, 0]
})

# Division by zero
df['result'] = df['numerator'] / df['denominator']
print(df)
# Expected output:
#    numerator  denominator  result
# 0         10            2     5.0
# 1        -10            0    -inf
# 2          0            0     NaN
# 3          5            0     inf
```

**Why this output:** `10 / 2 = 5.0`. `-10 / 0` produces `-inf`. `0 / 0` produces `NaN`. `5 / 0` produces `inf`. No exception is raised.

**Example 2: Replacing Infinity Values**

```python
# Replace inf and -inf with 0
df['result_clean'] = df['result'].replace([np.inf, -np.inf], 0)
print(df[['numerator', 'denominator', 'result', 'result_clean']])
# Expected output:
#    numerator  denominator  result  result_clean
# 0         10            2     5.0           5.0
# 1        -10            0    -inf           0.0
# 2          0            0     NaN           NaN
# 3          5            0     inf           0.0
```

**Why this output:** `.replace([np.inf, -np.inf], 0)` replaces both positive and negative infinity with 0. The `NaN` value remains unchanged.

**Example 3: Safe Division with np.divide**

```python
# Safe division using np.divide with where parameter
df['safe_division'] = np.divide(
    df['numerator'],
    df['denominator'],
    where=df['denominator'] != 0,
    out=np.zeros_like(df['numerator'], dtype=float)
)
print(df[['numerator', 'denominator', 'safe_division']])
# Expected output:
#    numerator  denominator  safe_division
# 0         10            2            5.0
# 1        -10            0            0.0
# 2          0            0            0.0
# 3          5            0            0.0
```

**Why this output:** The `where` parameter specifies the condition under which division occurs. Where `denominator == 0`, the `out` array (initialized to zeros) is used instead. This avoids `inf` and `NaN` entirely.

### Real-World Cases

- **Financial ratios:** Computing debt-to-equity ratios where some companies have zero equity.
- **Growth rates:** Calculating percentage growth where the base period is zero.
- **Sensor data:** Computing ratios where some sensors report zero readings.

### References

- Division by 0 in pandas — Stack Overflow — https://stackoverflow.com/questions/75311125
- Handling division by zero in Pandas calculations — Stack Overflow — https://stackoverflow.com/questions/45548771
- pandas conventions regarding division by zero — https://github.com/pandas-dev/pandas/blob/main/pandas/core/ops/missing.py


## 4. Absolute Values and Transformations: abs(), exp(), log(), sqrt()

### Definitions

**Core Definition:** Mathematical transformations such as `abs()`, `np.sqrt()`, `np.exp()`, and `np.log()` apply element-wise mathematical functions to entire Series or DataFrames.

**Technical Definition:** Pandas exposes `DataFrame.abs()` and `Series.abs()` as methods that return the absolute value of each element. NumPy universal functions (ufuncs) such as `np.sqrt()`, `np.exp()`, `np.log()`, `np.log10()`, `np.log1p()`, and `np.expm1()` can be applied directly to pandas objects, operating element-wise and returning pandas objects with the same index. These transformations are commonly used in feature engineering and statistical modeling.

**Beginner-Friendly Explanation:** These functions transform every number in a column. `abs()` makes negative numbers positive. `np.sqrt()` takes the square root. `np.exp()` computes e raised to each power. `np.log()` computes the natural logarithm. They're useful for scaling, normalizing, and analyzing data.

### Purposes

- To compute absolute values for magnitude-based analysis.
- To apply logarithmic transformations for reducing skewness in right-skewed data.
- To compute exponential transformations for growth modeling.
- To compute square roots for variance stabilization.
- To combine transformations in feature engineering pipelines.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
# Absolute value
df['col'].abs()
df[['A', 'B']].abs()

# Square root
import numpy as np
np.sqrt(df['col'])

# Exponential
np.exp(df['col'])

# Natural logarithm
np.log(df['col'])

# Base-10 logarithm
np.log10(df['col'])

# log(1 + x) — useful for small values
np.log1p(df['col'])

# exp(x) - 1 — inverse of log1p
np.expm1(df['col'])
```

**Component Breakdown:**

| Function | Description | Domain |
|----------|-------------|--------|
| `.abs()` | Absolute value | All numeric |
| `np.sqrt()` | Square root | x ≥ 0 |
| `np.exp()` | e^x | All numeric |
| `np.log()` | Natural log | x > 0 |
| `np.log10()` | Base-10 log | x > 0 |
| `np.log1p()` | log(1 + x) | x > -1 |
| `np.expm1()` | e^x − 1 | All numeric |

**Syntax Rules:**

- `.abs()` is a pandas method; the others are NumPy ufuncs applied to pandas objects.
- All functions operate element-wise and preserve the index.
- `np.log()` and `np.sqrt()` on negative values produce `NaN` and emit a runtime warning.
- `np.log1p()` is more accurate than `np.log(1 + x)` for small values of x.

**Constraints and Limitations:**

- `np.log(0)` produces `-inf`; `np.log(negative)` produces `NaN`.
- `np.sqrt(negative)` produces `NaN`.
- Transformations may produce `inf` or `NaN` that need to be handled.
- Applying NumPy ufuncs to nullable dtypes may return different results than for NumPy dtypes.

### Annotated Code Examples

**Example 1: Absolute Value and Square Root**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'value': [-4, -1, 0, 1, 4, 9]
})

df['abs_value'] = df['value'].abs()
df['sqrt_abs'] = np.sqrt(df['value'].abs())
print(df)
# Expected output:
#    value  abs_value  sqrt_abs
# 0     -4          4       2.0
# 1     -1          1       1.0
# 2      0          0       0.0
# 3      1          1       1.0
# 4      4          4       2.0
# 5      9          9       3.0
```

**Why this output:** `.abs()` converts negative values to positive. `np.sqrt()` is applied to the absolute values to avoid `NaN` from negative inputs.

**Example 2: Logarithmic and Exponential Transformations**

```python
# Logarithmic transformation
df2 = pd.DataFrame({'value': [1, 10, 100, 1000]})
df2['log_value'] = np.log(df2['value'])
df2['exp_log'] = np.exp(df2['log_value'])
print(df2)
# Expected output:
#    value  log_value  exp_log
# 0      1   0.000000      1.0
# 1     10   2.302585     10.0
# 2    100   4.605170    100.0
# 3   1000   6.907755   1000.0
```

**Why this output:** `np.log()` computes the natural logarithm. `np.exp()` is the inverse operation, recovering the original values. The round-trip `exp(log(x))` returns x.

**Example 3: Handling Invalid Domain Values**

```python
# Values outside the domain of log and sqrt
df3 = pd.DataFrame({'value': [-1, 0, 1, 2]})

# log of negative and zero
df3['log_value'] = np.log(df3['value'])
print(df3)
# Expected output:
#    value  log_value
# 0     -1        NaN
# 1      0       -inf
# 2      1        0.0
# 3      2        0.7
```

**Why this output:** `np.log(-1)` produces `NaN` because negative numbers are outside the domain. `np.log(0)` produces `-inf`. These special values should be handled before downstream analysis.

### Real-World Cases

- **Income data:** Apply `np.log()` to income columns to reduce right skewness before regression.
- **Population growth:** Use `np.exp()` to model exponential growth processes.
- **Signal processing:** Apply `.abs()` to extract magnitude from complex or signed signals.

### References

- pandas.DataFrame.abs — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.abs.html
- pandas.eval (mathematical functions) — https://pandas.pydata.org/docs/reference/api/pandas.eval.html
- numpy.log — https://numpy.org/doc/stable/reference/generated/numpy.log.html
- numpy.exp — https://numpy.org/doc/stable/reference/generated/numpy.exp.html
- numpy.sqrt — https://numpy.org/doc/stable/reference/generated/numpy.sqrt.html


## 5. Cumulative Variations: Running Trends with cumsum(), cumprod(), cummin(), cummax()

### Definitions

**Core Definition:** Cumulative operations compute running aggregates (sum, product, minimum, or maximum) along an axis, where each output value represents the aggregate of all values up to and including that position.

**Technical Definition:** `DataFrame.cumsum(axis=None, skipna=True, *args, **kwargs)` returns a DataFrame of the same shape where each element is the cumulative sum of the values from the beginning up to that element. `cumprod()`, `cummin()`, and `cummax()` perform analogous operations for product, minimum, and maximum. All methods have a `skipna` parameter (default `True`) that excludes `NaN` values from the computation while preserving their locations in the result. These methods are also available on Series and GroupBy objects.

**Beginner-Friendly Explanation:** Cumulative operations compute a running total. If you have daily sales of [100, 200, 50], the cumulative sum is [100, 300, 350]. Cumulative product multiplies as it goes. Cumulative min and max keep track of the smallest or largest value seen so far.

### Purposes

- To compute running totals for financial and operational analysis.
- To track running maximum or minimum values for drawdown analysis.
- To compute cumulative returns or compound growth.
- To create features for time-series models that capture trend information.
- To perform group-wise cumulative operations after `groupby()`.

### Syntax Rules and Structure

**Complete General Syntax:**

```python
DataFrame.cumsum(axis=None, skipna=True, *args, **kwargs)
DataFrame.cumprod(axis=None, skipna=True, *args, **kwargs)
DataFrame.cummin(axis=None, skipna=True, *args, **kwargs)
DataFrame.cummax(axis=None, skipna=True, *args, **kwargs)
Series.cumsum(axis=None, skipna=True, *args, **kwargs)
```

**Component Breakdown:**

| Method | Description |
|--------|-------------|
| `.cumsum()` | Cumulative sum |
| `.cumprod()` | Cumulative product |
| `.cummin()` | Cumulative minimum |
| `.cummax()` | Cumulative maximum |
| `axis` | `0` or `'index'`: down rows (default); `1` or `'columns'`: across columns |
| `skipna` | `True` (default): exclude NA values; `False`: include NA values |

**Syntax Rules:**

- All cumulative methods preserve the location of `NaN` values in the output.
- With `skipna=True` (default), `NaN` values are ignored but their positions are preserved as `NaN` in the result.
- With `skipna=False`, any `NaN` encountered causes all subsequent cumulative values to be `NaN`.
- Group-wise cumulative operations are available via `groupby().cumsum()`, etc.

**Constraints and Limitations:**

- Cumulative operations on integer columns may overflow for very large values.
- `cumprod()` on large datasets can produce extremely large numbers.
- The first element of the cumulative result is always the first element of the original data (unless it is `NaN`).

### Annotated Code Examples

**Example 1: Cumulative Sum**

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'sales': [100, 200, 50, 150, 75]
})

df['cumulative_sales'] = df['sales'].cumsum()
print(df)
# Expected output:
#    sales  cumulative_sales
# 0    100               100
# 1    200               300
# 2     50               350
# 3    150               500
# 4     75               575
```

**Why this output:** Each value in `cumulative_sales` is the sum of all previous sales values up to that row. Row 0 is 100, row 1 is 100+200=300, row 2 is 300+50=350, and so on.

**Example 2: Cumulative Product and Cumulative Max**

```python
df2 = pd.DataFrame({
    'returns': [0.05, 0.02, -0.03, 0.04],
    'prices': [10, 12, 8, 15]
})

# Cumulative product of (1 + returns) for compound growth
df2['cum_growth'] = (1 + df2['returns']).cumprod()
df2['cum_max_price'] = df2['prices'].cummax()
df2['cum_min_price'] = df2['prices'].cummin()
print(df2)
# Expected output:
#    returns  prices  cum_growth  cum_max_price  cum_min_price
# 0     0.05      10       1.050             10             10
# 1     0.02      12       1.071             12             10
# 2    -0.03       8       1.039             12              8
# 3     0.04      15       1.081             15              8
```

**Why this output:** `cumprod()` computes the running product of growth factors: 1.05, 1.05×1.02=1.071, 1.071×0.97=1.039, etc. `cummax()` tracks the highest price seen so far: 10, 12, 12, 15. `cummin()` tracks the lowest: 10, 10, 8, 8.

**Example 3: Cumulative Operations with NaN**

```python
df3 = pd.DataFrame({
    'value': [1, np.nan, 3, 4]
})

df3['cumsum_default'] = df3['value'].cumsum()
df3['cumsum_no_skip'] = df3['value'].cumsum(skipna=False)
print(df3)
# Expected output:
#    value  cumsum_default  cumsum_no_skip
# 0    1.0             1.0             1.0
# 1    NaN             NaN             NaN
# 2    3.0             4.0             NaN
# 3    4.0             8.0             NaN
```

**Why this output:** With `skipna=True` (default), the `NaN` is ignored but its position is preserved as `NaN`; the cumulative sum continues at row 2 with 1+3=4. With `skipna=False`, the `NaN` at row 1 causes all subsequent cumulative sums to be `NaN`.

### Real-World Cases

- **Financial analysis:** Compute cumulative returns or running profit/loss.
- **Inventory management:** Track cumulative inventory levels over time.
- **Time series features:** Create cumulative sum features for trend analysis.
- **Risk analysis:** Track running maximum drawdown using `cummax()`.

### References

- pandas.DataFrame.cumsum — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.cumsum.html
- pandas.DataFrame.cumprod — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.cumprod.html
- pandas.DataFrame.cummin — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.cummin.html
- pandas.DataFrame.cummax — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.cummax.html
- pandas.core.groupby.DataFrameGroupBy.cumsum — https://pandas.pydata.org/docs/reference/api/pandas.core.groupby.DataFrameGroupBy.cumsum.html
- Cumulative methods (User Guide) — https://pandas.pydata.org/docs/user_guide/basics.html#cumulative-methods


## References

- pandas.DataFrame.add — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.add.html
- pandas.DataFrame.sub — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.sub.html
- pandas.DataFrame.mul — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.mul.html
- pandas.DataFrame.div — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.div.html
- pandas.DataFrame.pow — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.pow.html
- pandas.DataFrame.abs — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.abs.html
- pandas.DataFrame.cumsum — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.cumsum.html
- pandas.DataFrame.cumprod — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.cumprod.html
- pandas.DataFrame.cummin — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.cummin.html
- pandas.DataFrame.cummax — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.cummax.html
- pandas.DataFrame.align — https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.align.html
- pandas.eval (mathematical functions) — https://pandas.pydata.org/docs/reference/api/pandas.eval.html
- Binary operations (User Guide) — https://pandas.pydata.org/docs/user_guide/basics.html#binary-operation
- Index alignment (User Guide) — https://pandas.pydata.org/docs/user_guide/basics.html#index-alignment
- Cumulative methods (User Guide) — https://pandas.pydata.org/docs/user_guide/basics.html#cumulative-methods
- Division by 0 in pandas — Stack Overflow — https://stackoverflow.com/questions/75311125
- Handling division by zero in Pandas calculations — Stack Overflow — https://stackoverflow.com/questions/45548771
- numpy.log — https://numpy.org/doc/stable/reference/generated/numpy.log.html
- numpy.exp — https://numpy.org/doc/stable/reference/generated/numpy.exp.html
- numpy.sqrt — https://numpy.org/doc/stable/reference/generated/numpy.sqrt.html